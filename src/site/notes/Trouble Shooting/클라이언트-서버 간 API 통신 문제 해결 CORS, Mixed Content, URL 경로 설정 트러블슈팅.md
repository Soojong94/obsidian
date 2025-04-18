---
{"dg-publish":true,"permalink":"/trouble-shooting/api-cors-mixed-content-url/"}
---


## 문제 상황

- 웹 브라우저에서 로그인 API 요청 시 타임아웃 발생
- 서버 내부에서 curl로 요청하면 정상 응답
- 브라우저에서 `ERR_CONNECTION_REFUSED` 오류 발생 후 `403 Forbidden`으로 변경
- "Mixed Content" 오류: HTTPS 페이지에서 HTTP API 요청 시도

## 서버 구성

- 웹서버: Nginx
- 백엔드: Java 애플리케이션 (8080 포트)
- 프론트엔드: Vue.js (5173 포트)

## 문제 원인

### Cross-Origin 요청 문제

- 브라우저에서 실행되는 JavaScript가 localhost:8080이나 하드코딩된 IP 주소로 직접 API 요청
- 브라우저의 다른 출처(Origin) 정책으로 인해 차단됨

### Mixed Content 문제

- HTTPS로 로드된 페이지에서 HTTP 요청을 시도할 때 브라우저가 차단
- 사용자는 HTTPS로 접속했으나, API 요청은 HTTP로 전송됨

### 하드코딩된 API 주소

- 프론트엔드 코드에서 API 요청 URL이 상대 경로가 아닌 절대 경로로 하드코딩됨
- 요청이 서버 IP 주소로 직접 전송됨

## 해결 방법

### 1. Nginx 설정 수정

```nginx
# 80 포트에서 443 포트로 리다이렉트
server {
    listen 80;
    server_name [도메인];
    return 301 https://$host$request_uri;
}

# 443 포트에서의 메인 서버 설정
server {
    listen 443 ssl;
    server_name [도메인];
    ssl_certificate [인증서 경로];
    ssl_certificate_key [키 파일 경로];
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    
    # API 요청을 백엔드로 프록시
    location /api/ {
        proxy_pass http://[내부IP]:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Origin https://[도메인];
        proxy_set_header Referer https://[도메인];
    }
    
    location / {
        proxy_pass http://127.0.0.1:5173;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# 5173 포트 서버 설정
server {
    listen 5173;
    server_name _;
    # 로그 수집 중지
    access_log off;
    error_log /dev/null;
    root [프론트엔드 경로];
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
    error_page 404 /index.html;
}
```

### 2. 프론트엔드 코드 수정

```javascript
// 변경 전 (문제가 있는 코드)
axios.post('http://[서버IP]:8080/api/v1/auth/login', data)

// 변경 후 (올바른 코드)
axios.post('/api/v1/auth/login', data)
```

### 3. 설정 및 코드 수정 후 테스트

Nginx 설정 저장 후 문법 검사 및 재시작:

```bash
nginx -t
systemctl reload nginx
```

문제 해결 확인 테스트:

```bash
curl -v -X POST http://[내부IP]:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -H "Origin: https://[도메인]" \
  -H "Referer: https://[도메인]" \
  -d '{"loginId":"[사용자ID]","password":"[비밀번호 해시]"}'
```

## 주요 개념 정리

### 브라우저의 Same-Origin 정책

- 브라우저는 보안상의 이유로, 다른 출처(도메인, 프로토콜, 포트가 다른 경우)에서의 요청을 제한함
- 이를 해결하기 위해 CORS(Cross-Origin Resource Sharing) 설정이 필요

### 상대 경로 vs 절대 경로

- 상대 경로(`/api/...`)는 현재 도메인을 기준으로 요청을 보냄
- 절대 경로(`http://서버주소/api/...`)는 지정된 주소로 직접 요청을 보냄
- 웹 애플리케이션에서는 상대 경로 사용이 권장됨

### Nginx 프록시의 역할

- 클라이언트 요청을 받아 적절한 백엔드 서버로 전달
- 동일 도메인에서 다양한 서비스(API, 정적 파일 등)를 제공 가능
- CORS 문제 해결에 도움이 됨

### localhost의 의미

- 브라우저에서 localhost는 사용자 컴퓨터를 의미
- 서버에서 localhost는 서버 자신을 의미
- 이로 인해 브라우저에서 localhost 주소를 하드코딩하면 문제 발생

### Mixed Content 이슈

- HTTPS 페이지에서 HTTP 리소스를 로드하려고 할 때 발생
- 브라우저는 보안상의 이유로 이를 차단함
- 해결책: 모든 리소스를 HTTPS로 제공하거나 상대 경로 사용

## 최종 요약

- 웹 애플리케이션에서 API 요청 시 하드코딩된 주소 대신 상대 경로 사용하기
- Nginx를 통해 API 요청을 백엔드 서버로 프록시하여 CORS 문제 해결
- 프론트엔드와 백엔드가 다른 서버에 있더라도 사용자에게는 하나의 도메인으로 통합 제공
- HTTPS 페이지에서는 HTTP 리소스 요청을 피하고, 상대 경로나 HTTPS URL 사용하기
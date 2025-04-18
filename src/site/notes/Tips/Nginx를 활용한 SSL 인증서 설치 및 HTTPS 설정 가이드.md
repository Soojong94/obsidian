---
{"dg-publish":true,"permalink":"/tips/nginx-ssl-https/"}
---



## 📌 개요

웹사이트를 안전하게 서비스하기 위한 HTTPS 설정 가이드입니다. SSL 인증서를 설치하고 Nginx 웹 서버를 설정하는 전체 과정을 안내합니다.

[[Tips/Web Application HTTPS Data Flow\|Web Application HTTPS Data Flow]]

**Nginx vs Apache 주요 차이점:**

- **Apache**: 더 오래된 전통적인 웹 서버로, 모듈 기반 구조와 .htaccess 파일을 통한 유연한 설정이 특징
- **Nginx**: 이벤트 기반 비동기 처리로 높은 동시접속에 강점이 있는 최신 웹 서버

## 🛠 사전 준비사항

- Ubuntu/Linux 서버 (AWS EC2, GCP 등의 클라우드 서버)
- 도메인 보유 (가비아, 후이즈 등에서 구매한 도메인)
- Node.js 애플리케이션을 3000번 포트 사용하여 구동한다고 가정

## 📝 단계별 설치 가이드

### 1. Nginx 웹 서버 설치

```bash
# 패키지 목록 업데이트
sudo apt update

# Nginx 웹 서버 설치
sudo apt install nginx

# Nginx 실행 상태 확인
sudo systemctl status nginx
```

### 2. SSL 인증서 발급 도구(Certbot) 설치 및 인증서 발급

**Certbot 설치**

```bash
# Certbot과 Nginx 플러그인 설치
sudo apt install certbot python3-certbot-nginx

# 설치된 Certbot 버전 확인
certbot --version
```

**SSL 인증서 발급**

```bash
# your-domain.com 부분을 실제 도메인으로 변경하여 실행
sudo certbot --nginx -d your-domain.com
```

💡 **인증서 발급 과정 안내:**

- 이메일 주소 입력: 인증서 만료 알림을 받을 이메일
- 약관 동의: Let's Encrypt 서비스 약관 (A)gree 선택
- 이메일 수신 여부: 선택사항
- HTTPS 리다이렉트: HTTP 접속을 HTTPS로 자동 전환할지 선택
    - 미설정시 http로 접속되며, 추후에 nginx 설정 변경 가능

### 3. Nginx 서버 설정

**설정 파일 생성**

```bash
# Nginx 설정 파일 생성 (your-domain.com을 실제 도메인으로 변경)
sudo nano /etc/nginx/sites-available/your-domain.com
```

**Nginx 설정 코드**

```nginx
# HTTP 서버 설정
server {
    listen 80;                    # 80번 포트(HTTP)로 들어오는 접속을 받음
    server_name your-domain.com;  # 설정을 적용할 도메인 이름

    # 모든 HTTP 요청을 HTTPS로 리다이렉트
    return 301 https://$server_name$request_uri;
}

# HTTPS 서버 설정
server {
    listen 443 ssl;                # 443번 포트(HTTPS)로 들어오는 접속을 받음
    server_name your-domain.com;   # 설정을 적용할 도메인 이름

    # SSL 인증서 파일 경로 설정
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;      # SSL 인증서 파일
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;    # SSL 인증서 키 파일

    # SSL 보안 설정
    ssl_protocols TLSv1.2 TLSv1.3;    # 사용할 SSL/TLS 프로토콜 버전
    ssl_prefer_server_ciphers on;      # 서버 측에서 선호하는 암호화 방식 사용
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;   # 사용할 암호화 알고리즘

    # Node.js 애플리케이션으로 요청 전달 설정
    location / {
        proxy_pass http://localhost:3000;   # Node.js 서버로 요청 전달 (포트번호 필요시 변경)

        # 프록시 서버 설정
        proxy_set_header Host $host;                    # 원래 요청의 도메인 정보 전달
        proxy_set_header X-Real-IP $remote_addr;        # 실제 방문자의 IP 주소 전달
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   # 프록시 서버를 거친 IP 정보
        proxy_set_header X-Forwarded-Proto $scheme;     # 원래 요청의 프로토콜(http/https) 전달
    }
}
```

**설정 활성화 및 적용**

```bash
# 설정 파일 심볼릭 링크 생성
sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/

# Nginx 설정 파일 문법 검사
sudo nginx -t

# Nginx 서버 재시작하여 설정 적용
sudo systemctl restart nginx
```

### 4. SSL 인증서 관리

**인증서 상태 확인**

```bash
# 설치된 SSL 인증서 정보 확인
sudo certbot certificates
```

**자동 갱신 설정 확인**

```bash
# 인증서 갱신 테스트 (실제로 갱신되지는 않음)
sudo certbot renew --dry-run

# 자동 갱신 서비스 동작 상태 확인
sudo systemctl status certbot.timer
```

## ✅ 설정 완료 후 확인사항

**접속 테스트 방법**

- 웹 브라우저에서 도메인으로 접속
    - http://도메인 입력 → https로 자동 전환되는지 확인
    - https://도메인 입력 → 보안 연결 확인
    - 브라우저의 자물쇠 아이콘 클릭하여 SSL 인증서 상태 확인
- 다음 주소로 접속하여 모두 동일한 페이지가 표시되는지 확인:
    - localhost:3000
    - 서버 IP 주소
    - 도메인 주소

## ⚠️ 주의사항

- Let's Encrypt SSL 인증서는 90일 후 만료됨 (자동 갱신 설정 필수)
- Node.js 애플리케이션이 3000번 포트에서 실행 중이어야 함
- 도메인의 DNS 설정이 서버 IP를 정확히 가리키고 있어야 함

## 🔍 문제 해결

**오류 발생 시 Nginx 로그 확인 방법:**

```bash
# Nginx 에러 로그 확인
sudo tail -f /var/log/nginx/error.log

# Nginx 접속 로그 확인
sudo tail -f /var/log/nginx/access.log
```

**문제 발생 시 체크리스트**

- 도메인 DNS 설정이 올바른지 확인
- 방화벽에서 80, 443 포트가 열려있는지 확인
- Node.js 애플리케이션이 정상 실행 중인지 확인
- Nginx 설정 파일 문법에 오류가 없는지 확인
- SSL 인증서 파일이 올바른 경로에 있는지 확인
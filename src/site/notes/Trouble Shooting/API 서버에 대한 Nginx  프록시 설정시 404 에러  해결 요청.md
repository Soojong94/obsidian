---
{"dg-publish":true,"permalink":"/Trouble Shooting/API 서버에 대한 Nginx  프록시 설정시 404 에러  해결 요청/"}
---





## 문제 상황

- API 서버(9001 포트)에 대한 Nginx 프록시 설정(80 포트) 시 404 에러 발생
- 80 포트로 들어오는 트래픽을 9001 포트로 포트포워딩 시도 중 실패

## 초기 상태

- API 서버: http://localhost:9001에서 정상 작동(inc 파일로 작성 후 include)
- Nginx: 80 포트로 프록시 시도 중 404 에러

## 문제의 설정 파일들

### /etc/nginx/sites-enabled/default (Ubuntu 기본 설정)

```bash
server {
    listen 80 default_server;          # IPv4 기본 80포트 리스닝
    listen [::]:80 default_server;     # IPv6 기본 80포트 리스닝

    root /var/www/html;                # 웹 루트 디렉토리 설정
    index index.html index.htm index.nginx-debian.html;  # 기본 인덱스 파일 설정
    server_name _;                     # 모든 호스트명 매칭

    location / {
        try_files $uri $uri/ =404;     # 파일 찾기 시도 후 404 반환
    }
}
```

### /etc/nginx/conf.d/default.conf (추가하려던 API 프록시 설정)

```bash
server {
    listen 80 default_server;          # IPv4 80포트 기본 서버 설정 (충돌 발생)
    listen [::]:80 default_server;     # IPv6 80포트 기본 서버 설정 (충돌 발생)
    server_name localhost;             # localhost 호스트명 매칭

    include /etc/nginx/conf.d/api-service-url.inc;  # API 서버 URL 설정 포함

    location /api/v1/ {                # API 경로 매칭
        proxy_pass $api_service_url;   # 설정된 API 서버로 프록시
        proxy_set_header X-Real-IP $remote_addr;           # 실제 클라이언트 IP 전달
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # 프록시 정보 전달
        proxy_set_header Host $http_host;                  # 원본 호스트 헤더 유지
    }
}
```

## 원인 분석

- **default_server 설정 충돌**
    - /etc/nginx/sites-enabled/default: 기본 80 포트 설정
    - /etc/nginx/conf.d/default.conf: API 프록시용 80 포트 설정
    - 동일 포트(80)에 대한 중복 default_server 설정 불가

## 기존 설정 파일 구조

```bash
# /etc/nginx/conf.d/api-service-url.inc
set $api_service_url http://127.0.0.1:9001;  # API 서버 주소 변수 설정

# /etc/nginx/conf.d/default.conf
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name localhost;

    include /etc/nginx/conf.d/api-service-url.inc;

    location /api/v1/ {
        proxy_pass $api_service_url;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
    }
}

# /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    # multi_accept on;
}

http {
    # 기본 설정
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # SSL 설정
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    # 로깅 설정
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip 압축
    gzip on;

    # 설정 파일 Include
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;  # 이 부분이 충돌의 원인
}
```

## 해결 방법

### 방법 1: nginx.conf에서 sites-enabled 설정 제외

```bash
http {
    include /etc/nginx/conf.d/*.conf;                # conf.d 디렉토리 설정 포함
    # include /etc/nginx/sites-enabled/*;            # 기본 설정 비활성화
}
```

### 방법 2: sites-enabled의 default 설정 제거

```bash
rm /etc/nginx/sites-enabled/default*
```

## 설정 테스트

```bash
rm /etc/nginx/sites-enabled/default*              # 기본 설정 파일 제거

sudo nginx -t                                     # 설정 문법 검사
sudo nginx -s reload                             # 설정 리로드

# 테스트 명령어
curl -v http://localhost:9001/api/v1/health/active-profile    # API 서버 직접 접근 테스트
curl -v http://localhost/api/v1/health/active-profile         # Nginx 프록시 경유 테스트
```

## Apache vs Nginx 설정 방식 비교

### Apache

- 순차적 설정 로드
- 후순위 설정이 선순위 설정 덮어쓰기
- 동일 포트 VirtualHost 중복 허용

### Nginx

- 명시적 충돌 방지 정책
- 동일 IP:포트의 default_server 중복 불허
- 설정 간 명확한 구분 강조
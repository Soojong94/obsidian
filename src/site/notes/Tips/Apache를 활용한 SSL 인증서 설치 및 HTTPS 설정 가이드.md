---
{"dg-publish":true,"permalink":"/Tips/Apache를 활용한 SSL 인증서 설치 및 HTTPS 설정 가이드/"}
---


## 📌 개요

Apache 웹 서버를 사용하여 웹사이트에 HTTPS를 설정하는 가이드입니다. Apache는 Nginx와 함께 가장 널리 사용되는 웹 서버 중 하나입니다.

[[Tips/Web Application HTTPS Data Flow\|Web Application HTTPS Data Flow]]

**Nginx vs Apache 주요 차이점:**

- **Apache**: 더 오래된 전통적인 웹 서버로, 모듈 기반 구조와 .htaccess 파일을 통한 유연한 설정이 특징
- **Nginx**: 이벤트 기반 비동기 처리로 높은 동시접속에 강점이 있는 최신 웹 서버

## 🛠 사전 준비사항

- Ubuntu/Linux 서버
- 도메인 보유
- Node.js 애플리케이션 (3000번 포트 사용)

## 📝 단계별 설치 가이드

### 1. Apache 웹 서버 설치

```bash
# 패키지 업데이트
sudo apt update

# Apache 설치 (apache2가 Ubuntu/Debian의 패키지명)
sudo apt install apache2

# Apache 서비스 상태 확인
sudo systemctl status apache2
```

### 2. SSL 모듈 활성화

```bash
# Apache SSL 모듈 활성화
sudo a2enmod ssl
sudo a2enmod rewrite  # URL 리다이렉트용 모듈

# Apache 재시작
sudo systemctl restart apache2
```

### 3. Certbot 설치 및 SSL 인증서 발급

```bash
# Apache용 Certbot 설치
sudo apt install certbot python3-certbot-apache

# SSL 인증서 발급
sudo certbot --apache -d your-domain.com
```

### 4. Apache 가상 호스트 설정

```apache
# /etc/apache2/sites-available/your-domain.com.conf 파일 생성

# HTTP 설정: 80번 포트로 들어오는 모든 요청 처리
<VirtualHost *:80>
    # 이 가상 호스트가 응답할 도메인 이름 설정
    ServerName your-domain.com

    # 모든 HTTP 요청을 HTTPS로 영구 리다이렉트 (301 리다이렉트)
    Redirect permanent / https://your-domain.com/
</VirtualHost>

# HTTPS 설정: 443번 포트(SSL)로 들어오는 모든 요청 처리
<VirtualHost *:443>
    # 이 가상 호스트가 응답할 도메인 이름 설정
    ServerName your-domain.com

    # SSL 암호화 기능 활성화
    SSLEngine on
    # SSL 인증서 체인 파일 경로 지정 (도메인 인증서 + 중간 인증서)
    SSLCertificateFile /etc/letsencrypt/live/your-domain.com/fullchain.pem
    # SSL 개인 키 파일 경로 지정
    SSLCertificateKeyFile /etc/letsencrypt/live/your-domain.com/privkey.pem

    # 원본 호스트 헤더 유지 (프록시 사용 시 중요)
    ProxyPreserveHost On
    # 모든 요청을 로컬호스트의 3000번 포트로 전달
    ProxyPass / http://localhost:3000/
    # 응답 헤더의 URL을 원래 요청된 도메인으로 재작성
    ProxyPassReverse / http://localhost:3000/

    # HSTS 설정: 브라우저가 항상 HTTPS를 사용하도록 강제 (1년 유효)
    Header always set Strict-Transport-Security "max-age=31536000"

    # 에러 로그 파일 위치 지정
    ErrorLog ${APACHE_LOG_DIR}/your-domain.com-error.log
    # 접근 로그 파일 위치 지정 (combined는 표준 로그 형식)
    CustomLog ${APACHE_LOG_DIR}/your-domain.com-access.log combined
    
</VirtualHost>
```

### 5. 설정 활성화

```bash
# 프록시 모듈 활성화
sudo a2enmod proxy
sudo a2enmod proxy_http

# 가상 호스트 설정 활성화 (sites-enabled에 심볼릭 링크 생성)
sudo a2ensite your-domain.com.conf

# 설정 문법 검사
sudo apache2ctl configtest

# Apache 재시작
sudo systemctl restart apache2
```

## ⚙️ 주요 Apache 용어 설명

- **VirtualHost**: 하나의 서버에서 여러 도메인을 호스팅할 수 있게 해주는 설정
- **SSLEngine**: SSL/TLS 암호화 기능을 켜고 끄는 스위치
- **ProxyPass**: 들어오는 요청을 다른 서버(여기서는 Node.js)로 전달하는 설정
- **a2enmod**: Apache 모듈을 활성화하는 Ubuntu/Debian 명령어
- **a2ensite**: Apache 가상 호스트 설정을 활성화하는 명령어 (자동으로 심볼릭 링크 생성)

## 🔍 문제 해결

```bash
# Apache 오류 로그 확인
sudo tail -f /var/log/apache2/error.log

# 가상 호스트별 오류 로그 확인
sudo tail -f /var/log/apache2/your-domain.com-error.log
```

## ✅ 설정 확인 및 주의사항

### Apache 관련 주요 명령어

```bash
# 사이트 활성화 (sites-enabled에 심볼릭 링크 생성)
sudo a2ensite your-domain.com.conf

# 사이트 비활성화 (심볼릭 링크 제거)
sudo a2dissite your-domain.com.conf

# 현재 활성화된 사이트 목록 확인
ls -l /etc/apache2/sites-enabled/
```

### ✅ 체크리스트

- mod_ssl, mod_proxy 모듈이 활성화되어 있는지 확인
- SSL 인증서 파일 권한이 올바른지 확인
- Apache 설정 파일의 문법이 정확한지 확인
- SELinux가 활성화된 경우 적절한 컨텍스트 설정 필요

Let's Encrypt SSL 인증서는 90일마다 자동으로 갱신되며, Apache는 자동으로 재시작됩니다.
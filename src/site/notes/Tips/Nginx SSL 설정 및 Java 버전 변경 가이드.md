---
{"dg-publish":true,"permalink":"/tips/nginx-ssl-java/"}
---


## 1. Nginx SSL 설정 (도메인 기반 HTTPS 설정)

```nginx
# 80 포트로 접근 시 HTTPS로 리다이렉트
server {
    listen 80;
    server_name [도메인];
    return 301 https://$host$request_uri;
}

# 443 포트(HTTPS) 설정 - 5173으로 프록시
server {
    listen 443 ssl;
    server_name [도메인];
    
    # SSL 인증서 설정
    ssl_certificate [인증서 경로]/fullchain.pem;
    ssl_certificate_key [인증서 경로]/privkey.pem;
    
    # SSL 보안 강화 설정
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    
    # 5173 포트로 모든 요청을 프록시
    location / {
        proxy_pass http://localhost:5173;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

# 5173 포트 설정 (실제 애플리케이션 서버)
server {
    listen 5173;
    server_name localhost;
    
    root [웹 파일 경로];
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    error_page 404 /index.html;
}

# IP 접속 차단을 위한 default 서버 설정
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    server_name _;
    
    # SSL 설정 (default_server에도 필요)
    ssl_certificate [인증서 경로]/fullchain.pem;
    ssl_certificate_key [인증서 경로]/privkey.pem;
    
    # IP로 접속 시 연결 종료
    return 444;
}
```

### 사용법

1. `/etc/nginx/conf.d/` 디렉토리에 설정 파일 생성
2. 설정 파일에 위 내용 복사 후 도메인 및 경로 수정
3. 설정 검증 및 적용:

```bash
nginx -t
systemctl restart nginx
```

## 2. Java 버전 변경 (JDK 8 → JDK 17)

### 기존 설정 백업

```bash
# alternatives 설정 백업
alternatives --display java > /root/java_alternatives_backup_$(date +%Y%m%d).txt

# 환경 변수 설정 백업
if [ -f /etc/profile.d/jdk.sh ]; then
    cp /etc/profile.d/jdk.sh /root/jdk_sh_backup_$(date +%Y%m%d).sh
fi

# JAVA_HOME 참조 검색 및 백업
grep -r "JAVA_HOME" /etc/ > /root/java_home_references_$(date +%Y%m%d).txt
```

### Java 17 설정

```bash
# 확인: Java 17 설치 경로
ls -la [설치 경로] | grep jdk

# Java 17 alternatives 설정
alternatives --install /usr/bin/java java [JDK 17 경로]/bin/java 2000000
alternatives --set java [JDK 17 경로]/bin/java

# 환경 변수 설정
echo '# Java 17 환경 변수 설정' > /etc/profile.d/jdk.sh
echo 'export JAVA_HOME=[JDK 17 경로]' >> /etc/profile.d/jdk.sh
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> /etc/profile.d/jdk.sh
chmod +x /etc/profile.d/jdk.sh

# 현재 세션에 적용
source /etc/profile.d/jdk.sh
```

### 설정 확인

```bash
# Java 버전 확인
java -version

# 환경 변수 확인
echo $JAVA_HOME

# alternatives 설정 확인
alternatives --display java
```

### 설명

- `alternatives` 시스템은 여러 버전의 Java를 관리합니다. 우선순위가 높은 버전(숫자가 큰)이 자동 모드에서 선택됩니다.
- `/etc/profile.d/jdk.sh` 파일은 모든 사용자에게 적용되는 환경 변수 설정입니다.
- 서버 재시작 후에도 설정이 유지됩니다.

### 참고사항

- Nginx 로그 삭제: `cat /dev/null > [로그 경로]`
- Nginx 로그 수집 중지: `access_log off;` 및 `error_log /dev/null;`
- 방화벽 설정 확인: `firewall-cmd --list-all`
- DNS 설정이 올바른지 확인: `nslookup [도메인]`
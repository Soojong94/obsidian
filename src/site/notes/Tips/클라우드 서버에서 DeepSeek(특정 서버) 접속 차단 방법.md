---
{"dg-publish":true,"permalink":"/tips/deep-seek/"}
---


## 🛠️ 두 가지 주요 차단 방법

### 1. hosts 파일 수정

deepseek 접속 요청이 모두 localhost로 resolve 되도록 설정 deepseek ip는 nslookup 명령어를 통해 확인 가능

```
127.0.0.1    www.deepseek.com
127.0.0.1    www.deepseek.com.cdn.cloudflare.net
127.0.0.1    deepseek.com
127.0.0.1    104.18.27.90
127.0.0.1    104.18.26.90
```

**위치**:

- Windows: C:\Windows\System32\drivers\etc\hosts
- Linux: /etc/hosts

**주의사항**:

- Windows에서는 관리자 권한으로 수정 필요
- DNS 캐시 초기화 필요 (ipconfig /flushdns)
- 모든 가능한 접근 경로(도메인, 서브도메인, IP)를 차단해야 함

### 2. 방화벽 아웃바운드 규칙 설정

```
규칙 이름: Block DeepSeek
방향: 아웃바운드
작업: 연결 차단
프로토콜: TCP, UDP
포트: 모든 포트
원격 IP 주소: 104.18.26.90, 104.18.27.90
원격 도메인: www.deepseek.com, deepseek.com, www.deepseek.com.cdn.cloudflare.net
```

## 📝 권장 작업 순서

1. 현재 설정 백업 (hosts 파일, 방화벽 규칙)
2. hosts 파일 수정
3. DNS 캐시 초기화
4. 방화벽 아웃바운드 규칙 추가
5. 접속 차단 테스트 및 확인
6. 문제 발생 시 원상 복구 절차 실행

## ⚠️ 제한사항 및 주의점

- 우회 경로를 통한 접근까지 차단하는 것은 불가능함
- DeepSeek IP 정보 변경 시 일시적으로 접근 가능할 수 있음
- 브라우저 캐시 삭제가 필요할 수 있음

## 🔄 원상 복구 방법

1. hosts 파일 백업본 복원
2. 방화벽 아웃바운드 규칙 제거
3. DNS 캐시 초기화 (ipconfig /flushdns)

방화벽만 막았을 시 Time out error  
hosts 파일 설정을 통한 이중 차단 적용시
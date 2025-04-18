---
{"dg-publish":true,"permalink":"/Tips/Linux Server에 scp로 파일 옮기기/"}
---



※ MobaXterm과 같은 SSH를 이용해서 옮기는 방법도 쉽고 좋지만, 많은 서버를 관리한다면 PowerShell 등에서 alias 기능을 활용하여 접속하고 파일을 전송하는 것도 좋습니다. 
[[Tips/대량 서버 관리 1 - PowerShell Profile(alias) 만들기\|대량 서버 관리 1 - PowerShell Profile(alias) 만들기]]

## 1. 사전 준비사항

- 윈도우 로컬 PC에서 tar 파일 준비 (ex : app.tar.Z)
- 전송할 때 해당 파일이 현재 디렉토리에 위치 해 있는지 확인
- Linux 서버 접속 정보
    - IP 주소
    - 포트 번호 (포트포워딩 되어 있다면 외부 접속포트 입력)
    - 사용자 계정
    - 인증 방식 (키 페어 .pem 파일) - 비밀번호 인증 방식으로 접속할 시 생략

## 2. 파일 전송 절차

### 2.1 접속 테스트

```bash
# SSH 접속테스트(윈도우 환경에서)
ssh -i "[pem키경로]" -p [포트번호] [사용자명]@[IP주소]
```

### 2.2 SCP를 이용한 파일 전송

```bash
# 기본 명령어 구조
scp -P [포트번호] -i "[pem키경로]" [전송할파일명] [사용자명]@[IP주소]:[전송위치]

# 실제 예시
scp -P 22222 -i "C:/keys/ex1-key.pem" app.tar.Z admin@10.0.0.100:/home/admin/

# 여러 파일 전송
scp -P 22222 -i "C:/keys/ex1-key.pem" ex1.txt ex2.txt admin@10.0.0.100:/home/admin/

# 폴더 전체 전송 (-r 옵션)
scp -P 22222 -r -i "C:/keys/ex1-key.pem" ./folder admin@10.0.0.100:/home/admin/
```

## 3. 주의사항

- 포트 번호 옵션은 대문자 P 사용
    - scp -P 22222 (O) vs scp -p 22222 (X)
- 경로에 공백이 있는 경우 따옴표로 묶기
    - scp "C:/Program Files/test.txt" (O) vs scp C:/Program Files/test.txt (X)
- 권한 설정 확인
    - chmod 400 dev-key.pem (키 파일은 본인만 읽기 권한 필요)
- 방화벽 포트 개방 확인
    - SSH 접속 성공 여부로 포트 개방 확인
    - Test-NetConnection, namp, telnet 등을 사용한 불필요한 포트 스캔은 지양
- 접속 후 서버에서 확인 (필요한 경우):
    - sudo netstat -tulpn | grep [포트번호]

## 4. 문제 해결

- Permission denied 에러
    - Permission denied (publickey) → 키 파일 권한 확인 또는 잘못된 사용자 계정
- 타임아웃 에러
    - Connection timed out → 방화벽 확인 또는 잘못된 포트 번호
- 경로 관련 에러
    - No such file or directory → 로컬/원격 경로 존재 여부 확인

## 5. 참고사항

- root 디렉토리 vs 사용자 디렉토리
    - /root/file.txt (root) vs /home/admin/file.txt (사용자)
- 보안 관련 고려사항
    - ssh -vvv 옵션으로 상세 로그 확인하여 보안 문제 디버깅
- 대체 전송 방법 (SFTP)
    - sftp -P 22222 -i key.pem admin@10.0.0.100 (대화형 파일 전송)

## 6. 실제 사용 예시

```bash
# 필요한 부분 변경해서 사용 : key 위치, 파일명, 사용자명, ip 주소, 전송 위치

# 1. SSH 접속 테스트
ssh -i "C:/keys/ex1-key.pem" -p 22222 admin@10.0.0.100
Last login: Mon Nov 18 14:19:19 2024 from 192.168.1.100

# 2. 파일 전송 (tar 파일)
scp -P 22222 -i "C:/keys/ex1-key.pem" app.tar.Z admin@10.0.0.100:/home/admin/

# 3. alias 설정 후 사용 예시
# PowerShell에서
PS C:/Users/User> ssh-ex1
PS C:/Users/User> scp-ex1 "app.tar.Z"

# 4. SFTP 사용 예시
sftp -P 22222 -i "C:/keys/ex1-key.pem" admin@10.0.0.100
sftp> put app.tar.Z
sftp> ls -l
sftp> exit
```
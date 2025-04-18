---
{"dg-publish":true,"permalink":"/tips/1-power-shell-profile-alias/"}
---



PowerShell alias 설정 방법(ssh / scp 예시)

Powershell

```
# 1. PowerShell 프로필 설정
$PROFILE  # 프로필 경로 확인
New-Item -Path $PROFILE -Type File -Force  # 프로필 파일 없다면 생성
notepad $PROFILE  # 프로필 편집

# 2. alias 함수 추가 예시
## SSH 접속용 함수
function ssh-ex1 {
   ssh -i "C:/keys/ex1-key.pem" -p 22222 admin@10.0.0.100
}

function ssh-ex2 {
   ssh -i "C:/keys/ex2-key.pem" -p 22222 admin@10.0.0.200
}

function ssh-ex3 {
   ssh -i "C:/keys/ex3-key.pem" -p 22222 admin@10.0.0.300
}

## SCP 전송용 함수
function scp-ex1 {
   param($file)
   scp -P 22222 -i "C:/keys/ex1-key.pem" $file admin@10.0.0.100:/home/admin/
}

# 3. 프로필 적용
. $PROFILE

# 4. 사용 방법
ssh-ex1  # SSH 접속
scp-ex1 "파일명"  # 파일 전송
```

실행 예시

프로필 설정

![](https://beta.appflowy.cloud/api/file_storage/9694f649-00d1-4d32-8f94-e01c6e535655/v1/blob/304418fc%2De60e%2D49cd%2D8836%2D02eeb5e3d845/vsqccUeZyPOB9bTh8YVb7-nUik-Ox7vGVNI3JjWfeaQ=.png)

프로필 파일 생성

![](https://beta.appflowy.cloud/api/file_storage/9694f649-00d1-4d32-8f94-e01c6e535655/v1/blob/304418fc%2De60e%2D49cd%2D8836%2D02eeb5e3d845/mdRy3wQkAqQmTgyEb9dQMV5C09X9FTPrMbx8G6G7DyE=.png)

프로필 파일 작성

![](https://beta.appflowy.cloud/api/file_storage/9694f649-00d1-4d32-8f94-e01c6e535655/v1/blob/304418fc%2De60e%2D49cd%2D8836%2D02eeb5e3d845/4N-Ct4DTybAoTSi4f_DFbvSRuOmQ2cLsc-6f-Kgrfd0=.png)

적용

![](https://beta.appflowy.cloud/api/file_storage/9694f649-00d1-4d32-8f94-e01c6e535655/v1/blob/304418fc%2De60e%2D49cd%2D8836%2D02eeb5e3d845/Iab1NOLY1ENiqXh-3grr4SQ0sGTCM_I-ym5huu6qPcg=.png)

같은 원리로 Bastion Host 서버의 .bashrc 나 .profile 에 적용 가능

Bash

```
# Linux 서버 설정 (.bashrc 또는 .profile)

# SSH alias
alias ssh-ex1='ssh -i ~/.ssh/ex1-key.pem -p 22222 admin@10.0.0.100'
alias ssh-ex2='ssh -i ~/.ssh/ex2-key.pem -p 22222 admin@10.0.0.200' 
alias ssh-ex3='ssh -i ~/.ssh/ex3-key.pem -p 22222 admin@10.0.0.300'

# SCP alias 
alias scp-ex1='scp -P 22222 -i ~/.ssh/ex1-key.pem'
alias scp-ex2='scp -P 22222 -i ~/.ssh/ex2-key.pem'
alias scp-ex3='scp -P 22222 -i ~/.ssh/ex3-key.pem'

# 사용 예시
ssh-ex1  # 서버 접속
scp-ex1 local_file.txt admin@10.0.0.100:/home/admin/  # 파일 전송

source ~/.bashrc # 설정 적용 (또는 새로운 세션에서 로그인)
```
---
{"dg-publish":true,"permalink":"/trouble-shooting/kt-cloud-waf-pro-vm-key-pair/"}
---



## 문제 상황

- WAF_Pro VM 생성 후 초기 발급된 key pair로 ubuntu 계정 접속 불가
- 초기 비밀번호 미설정 상태

## 원인

- KT Server 초기 접속은 OS별 기본 계정(ubuntu/cloud-user) 사용 필요
- WAF는 ubuntu 계정으로 생성되었으나 초기 발급된 key pair로 로그인 불가
- 초기 비밀번호 미설정으로 서버 재생성 또는 싱글모드 접근 필요 

> [!NOTE]
>   싱글모드 : 루트 사용자만 로그인 가능하게 부팅 (접속 시 비밀번호 불필요, 윈도우 안전모드와 유사)


## 해결 과정

1. **싱글모드 접근**
    
    - WAF 방화벽 설정 완료 상태로 싱글모드 접근이 효율적
    - KT Tech Center에 싱글모드 접근 요청 (요청서 작성)
    - root 계정 로그인 허용 및 임시 비밀번호 수령
    - 임시 비밀번호로 서버 진입하여 ubuntu 계정 비밀번호 설정
2. **Key pair 로그인 설정**
    
    ```bash
    # authorized_keys 파일 확인
    ls -la ~/.ssh/authorized_keys
    cat ~/.ssh/authorized_keys
    
    # 기존 공개키 백업
    cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.backup
    
    # 로컬 PC에서 새 공개키 생성
    ssh-keygen -y -f my_key.pem > new_public_key.pub
    
    # 공개키 적용
    scp new_public_key.pub ubuntu@'WAF_IP':~/.ssh/
    cat ~/.ssh/new_public_key.pub > ~/.ssh/authorized_keys
    
    # 권한 설정
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    
    # Ubuntu 계정 로그인 테스트
    ssh -i my_key.pem ubuntu@'WAF_IP'
    ```
    

## 추가 설정

1. **관리자 계정 설정**
    
    - sudo 권한을 가진 관리자용 ID 생성
    - 비밀번호 로그인 활성화
    - sudoers 파일에 권한 관리 (대규모 계정 관리 시 wheel 그룹 활용)
    
2. **서버 보안 설정**
    
    ```bash
    # root SSH 접속 차단
    PermitRootLogin no
    
    # 계정별 인증 설정
    Match User ubuntu
        PasswordAuthentication no
        
    Match User admin_user
        PasswordAuthentication yes
    
    # sshd 재시작
    sudo systemctl restart sshd
    ```
    
3. **네트워크 설정**
    
    - Static NAT IP 사용으로 SSH 포트 변경 불가
    - 공인 IP 사용 시 포트포워딩 검토 필요
    
4. **최종 확인**
    
    - 서버 reboot 후 접속 테스트 완료
    - ubuntu 계정 비밀번호 인증 비활성화
    - 관리자 계정 인증 활성화 확인
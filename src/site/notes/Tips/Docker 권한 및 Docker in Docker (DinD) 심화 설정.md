---
{"dg-publish":true,"permalink":"/Tips/Docker 권한 및 Docker in Docker (DinD) 심화 설정/"}
---


## 1. Docker 권한 문제 해결

### 1.1 Jenkins 컨테이너 내부 권한 설정

```bash
# root 권한으로 Jenkins 컨테이너 접속
docker exec -it --user root jenkins bash

# 필요한 패키지 설치 및 권한 설정
apt-get update
apt-get install -y sudo
groupadd -f docker
usermod -aG docker jenkins

# 컨테이너 나가기
exit

# Jenkins 컨테이너 재시작
docker restart jenkins
```

### 1.2 호스트 시스템 Docker 소켓 권한 설정

```bash
# Docker 소켓 권한 변경
sudo chmod 666 /var/run/docker.sock
```

## 2. Docker in Docker (DinD) 상세 설명

### 2.1 DinD 구성 방식

**소켓 마운트 방식**

- 호스트의 Docker 데몬 사용
- `-v /var/run/docker.sock:/var/run/docker.sock` 옵션으로 구현
- 장점: 리소스 효율적, 설정 간단
- 단점: 호스트의 Docker 데몬을 공유하므로 완전한 격리가 안됨

**DinD 방식**

```bash
docker run -d \
  --name jenkins-dind \
  --privileged \
  -p 8080:8080 \
  -p 50000:50000 \
  docker:dind
```

- 컨테이너 내부에 별도 Docker 데몬 실행
- 장점: 보안 취약성 존재(privileged 권한)
- 단점: 리소스 소비 큼, 복잡한 설정 필요

### 2.2 보안 강화를 위한 권한 제한

```bash
# Docker 그룹 권한 제한
sudo chown root:docker /usr/bin/docker
sudo chmod 750 /usr/bin/docker

# Docker 소켓 권한 제한
sudo chown root:docker /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
```

## 3. 문제 해결 및 디버깅

### 3.1 권한 문제 디버깅

```bash
# Jenkins 사용자 권한 확인
docker exec jenkins whoami
docker exec jenkins groups

# Docker 소켓 권한 확인
ls -l /var/run/docker.sock

# Docker 명령어 테스트
docker exec jenkins docker ps
```

### 3.2 로그 확인

```bash
# Jenkins 컨테이너 로그 확인
docker logs jenkins

# Docker 데몬 로그 확인
sudo journalctl -u docker
```

## 4. 보안 고려사항

**최소 권한 원칙**

- 필요한 최소한의 권한만 부여
- 주기적인 권한 감사 실시

**네트워크 격리**

```bash
# 전용 네트워크 생성
docker network create jenkins-network

# 네트워크 적용
docker run -d \
  --name jenkins \
  --network jenkins-network \
```

**볼륨 마운트 제한**

- 필요한 디렉토리만 선택적으로 마운트
- 중요 시스템 디렉토리 마운트 제한
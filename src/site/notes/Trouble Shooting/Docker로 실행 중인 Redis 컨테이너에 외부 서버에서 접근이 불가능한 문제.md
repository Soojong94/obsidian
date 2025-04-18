---
{"dg-publish":true,"permalink":"/trouble-shooting/docker-redis/"}
---

# Redis 네트워크 연결 문제 해결


## 문제 상황

- user 서버에서 static 서버의 Redis로 접근 불가 (동일 서브넷 환경)
- static 서버의 Redis는 0.0.0.0으로 바인딩
- 로컬 redis-cli 테스트는 정상
- 외부(user 서버)에서 telnet/redis-cli 연결 실패

![](https://beta.appflowy.cloud/api/file_storage/9694f649-00d1-4d32-8f94-e01c6e535655/v1/blob/ce00170f%2D384c%2D4b0e%2D91c8%2Db456215e334e/vnmYwjjnL--JdY6944nhYF0RbQwOLhHKwgxwoneZ1Z0=.png)

## 원인

- Docker 기본 bridge 네트워크 모드 사용으로 컨테이너가 독립 네트워크 네임스페이스 보유
- bridge 모드에서 172.17.0.0/16 대역 IP 할당 및 NAT 통신
- 외부 접근 시 Docker NAT와 포트 매핑 과정에서 연결성 문제 발생

## 해결 과정

1. **현재 네트워크 상태 확인**

```bash
sudo iptables -L -n
sudo iptables -L -n -t nat
ip addr show docker0
bridge link show
```

2. **Docker 컨테이너 상태 확인**

```bash
docker ps
netstat -tulnp | grep 6379
```

3. **Redis 컨테이너 설정 확인**

```bash
docker logs redis
docker inspect redis
```

4. **Docker host 네트워크 모드로 재설정**

```bash
# 기존 컨테이너 제거
docker stop redis
docker rm redis

# host 네트워크 모드로 재실행
docker run --network host --name redis -d redis:latest --requirepass "password" --bind 0.0.0.0
```

5. **연결 테스트**

```bash
# Redis CLI 로컬 연결 테스트
redis-cli -h localhost -a password ping

# 포트 응답 확인
curl localhost:6379

# 원격 서버 연결 테스트
telnet static_server_ip 6379
```

## 추가 설정

```bash
# Docker 컨테이너 자동 실행 설정
docker update --restart=always redis

# Docker 로그 확인
docker logs -f redis

# Redis 컨테이너 상태 모니터링
docker stats redis
```

## 네트워크 구성 변경 사항

### 변경 전

- Docker bridge 네트워크 (172.17.0.0/16) 사용
- Redis 컨테이너에 172.17.0.2 IP 할당
- 호스트 6379 포트를 컨테이너 6379 포트로 포워딩
- NAT를 통한 외부 통신

### 변경 후

- Docker host 네트워크 모드 사용
- 호스트의 네트워크 스택 직접 사용
- 네트워크 가상화 없이 호스트 IP/포트 직접 사용
- NAT/포트 포워딩 없는 직접 통신
- 호스트 네트워크 인터페이스 직접 사용으로 연결성 확보
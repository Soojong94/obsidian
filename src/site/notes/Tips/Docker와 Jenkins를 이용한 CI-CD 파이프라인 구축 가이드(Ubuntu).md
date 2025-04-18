---
{"dg-publish":true,"permalink":"/Tips/Docker와 Jenkins를 이용한 CI-CD 파이프라인 구축 가이드(Ubuntu)/"}
---

# CI/CD 간단 구성 가이드

## 1. 기본 환경 설정 (Ubuntu 기준 Docker & Jenkins 설치)

```bash
# Docker 설치
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# sudo를 붙이지 않고 docker 명령어 사용하기 위함
sudo usermod -aG docker [username]

# Jenkins 컨테이너 실행
sudo docker run -d \
  # 컨테이너 이름 지정
  --name jenkins \
  
  # 컨테이너에 추가 권한 부여, 편의상 privileged 사용 
  # Docker outside of Docker에서는 불필요한 설정 <-> Docker in Docker
  --privileged \
  
  # Jenkins 웹 인터페이스용 포트 매핑
  -p 8080:8080 \
  
  # Jenkins 에이전트 연결용 포트 매핑
  -p 50000:50000 \
  
  # Jenkins 데이터 영구 저장을 위한 볼륨 마운트
  -v jenkins_home:/var/jenkins_home \
  
  # Docker 소켓 파일 마운트(Docker outside of Docker를 위한 소켓)
  -v /var/run/docker.sock:/var/run/docker.sock \
  
  # Docker 실행 파일 마운트
  -v $(which docker):/usr/bin/docker \
  
  # Jenkins LTS 버전 이미지 사용
  jenkins/jenkins:lts
  
# Jenkins 컨테이너 내부에서 docker 그룹 권한 설정
sudo docker exec -u 0 jenkins bash -c "\
  groupadd -g $(getent group docker | cut -d: -f3) docker && \
  usermod -aG docker jenkins"

sudo docker restart jenkins

# Docker 소켓 권한 변경
sudo chmod 666 /var/run/docker.sock

# (프로덕션 환경에서는 권한을 제한적으로 사용 : 
# ex - 그룹 소유권을 docker로 변경 
# sudo chown root:docker /var/run/docker.sock
# sudo chmod 660 /var/run/docker.sock)
```

### 추가 설명

**Docker 소켓 및 실행 파일 마운트 부분**

1. `/var/run/docker.sock`: Docker 데몬과 통신하기 위한 소켓 파일. 이를 Jenkins 컨테이너에 마운트함으로써, Jenkins 컨테이너 내부에서 호스트의 Docker 데몬을 사용할 수 있게 됨.
    
2. `$(which docker):/usr/bin/docker`: 호스트의 Docker 실행 파일을 Jenkins 컨테이너 내부의 `/usr/bin/docker` 위치에 마운트 시킴. 이를 통해 Jenkins 컨테이너 내부에서 Docker 명령어를 실행할 수 있게 됨. `$(which docker)`를 사용하면 docker 실행 파일의 정확한 경로를 자동으로 찾아줌
    

**Docker 그룹 권한 설정 부분**

1. `docker exec -u 0`: root 권한으로 Jenkins 컨테이너 내부에서 명령을 실행. (`-u 0`은 root 사용자를 의미)
2. `groupadd -g $(getent group docker | cut -d: -f3) docker`: 호스트의 docker 그룹과 동일한 GID(Group ID)를 가진 docker 그룹을 컨테이너 내부에 생성.
3. `usermod -aG docker jenkins`: jenkins 사용자를 docker 그룹에 추가. 이를 통해 jenkins 사용자가 Docker 명령어를 실행할 수 있는 권한을 얻게 됨.

※ Docker 권한 및 Docker in Docker (DinD) 심화 설정

## 2. 프로젝트 구조 설정(깃허브 작업)

main branch의 nginx-test/ (예시) 아래에 작성

![Pasted image 20250418130906.png](/img/user/images/Pasted%20image%2020250418130906.png)
### Dockerfile

```dockerfile
# nginx 알파인 리눅스 버전을 기본 이미지로 사용
FROM nginx:alpine

# 로컬의 html 디렉토리를 nginx의 기본 웹 루트 디렉토리로 복사
COPY html /usr/share/nginx/html

# 컨테이너의 80번 포트를 외부에 노출
EXPOSE 80
```

### html/index.html (nginx-test/html/ 아래에 작성)

```html
<!DOCTYPE html>
<html>
<head>
    <!-- UTF-8 인코딩 설정으로 한글 깨짐 방지 -->
    <meta charset="UTF-8">
    <title>Nginx Test</title>
</head>
<body>
    <!-- 테스트용 웹페이지 내용 -->
    <h1>Hello from Nginx</h1>
    <p>테스트 개발 빌드입니다</p>
    <p>This is a test page deployed via Jenkins pipeline.</p>
</body>
</html>
```

### Jenkinsfile (main branch의 nginx-test/ 아래에 작성)

```groovy
// 파이프라인 정의 시작
pipeline {
    // 모든 에이전트에서 실행 가능하도록 설정
    agent any
    
    // 환경 변수 설정
    environment {
        // Docker 이미지 이름 설정
        DOCKER_IMAGE = "my-nginx"
        // 빌드 번호를 태그로 사용
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    // 파이프라인 단계 정의
    stages {
        // Git 저장소에서 코드 체크아웃
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/[username]/[repository].git'
            }
        }
        
        // Docker 이미지 빌드
        stage('Build') {
            steps {
                // nginx-test 디렉토리로 이동
                dir('nginx-test') {
                    // Docker 이미지 빌드 명령 실행
                    sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
                }
            }
        }
        
        // 새 버전 배포
        stage('Deploy') {
            steps {
                sh '''
                    # 기존 컨테이너 중지 (실패해도 계속 진행)
                    docker stop nginx-app || true
                    # 기존 컨테이너 제거
                    docker rm nginx-app || true
                    # 새 버전 컨테이너 실행
                    docker run -d --name nginx-app -p 8090:80 $DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }
    }
    
    // 파이프라인 완료 후 실행될 작업
    post {
        // 성공 시 메시지 출력
        success {
            echo 'Pipeline succeeded! Nginx is deployed.'
        }
        // 실패 시 메시지 출력
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}
```

## 3. Jenkins 설정

- Jenkins 초기 비밀번호는 `docker logs jenkins` 명령어로 확인
- 필요한 플러그인 설치 (Git plugin, Docker Pipeline) - 권장 플러그인으로 설치해도 무방함
- New item - 새로운 Pipeline 프로젝트 생성
- 사진 참고하여 설정 진행 - url, branch, path 등은 상황에 맞게 변경
![Pasted image 20250418130959.png](/img/user/images/Pasted%20image%2020250418130959.png)
![Pasted image 20250418130936.png](/img/user/images/Pasted%20image%2020250418130936.png)
![Pasted image 20250418130939.png](/img/user/images/Pasted%20image%2020250418130939.png)

## 4. GitHub Webhook 설정

GitHub 저장소 Settings > Webhooks > Add webhook

- Payload URL: `http://[jenkins-서버-주소]:8080/github-webhook/` (Jenkins 서버로 웹훅을 전송할 URL)
- Content type: application/json (웹훅 데이터 형식 지정)
- SSL verification: Disable (테스트 환경용) (개발 환경에서는 SSL 검증 비활성화)
- Just the push event 선택 (push 이벤트가 발생할 때만 웹훅 전송)

## 5. 접속 및 테스트

- Jenkins 대시보드: `http://[서버-주소]:8080`
- 배포된 nginx 페이지: `http://[서버-주소]:8090`

## 작동 원리

1. GitHub에 코드 변경사항 push
2. GitHub webhook이 Jenkins에 알림
3. Jenkins가 코드를 가져와서 Docker 이미지 빌드
4. 기존 컨테이너 중지 및 제거
5. 새로운 버전의 컨테이너 배포
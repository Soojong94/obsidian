---
{"dg-publish":true,"permalink":"/kubernetes/pod/"}
---


Pod는 쿠버네티스에서 생성하고 관리할 수 있는 가장 작은 배포 단위입니다. 하나 이상의 컨테이너를 포함하며, 이들은 스토리지와 네트워크를 공유합니다.

## 특징

- 동일한 Pod 내의 컨테이너는 같은 네트워크 네임스페이스를 공유하며 서로 localhost로 통신 가능
- 항상 같은 노드에서 함께 실행되며 함께 스케줄링됨
- 동일한 Pod의 컨테이너들은 볼륨을 공유 가능
- 각 Pod는 고유한 IP 주소를 가짐
- Pod는 임시적(ephemeral)이며 언제든지 삭제될 수 있음
- 재시작 정책(Always, OnFailure, Never)을 통해 컨테이너 장애 처리

## 클라우드 네트워크 개념과 비교

- **VM/서버 비교**: 전통적인 클라우드 인프라에서 VM이나 인스턴스가 기본 단위라면, 쿠버네티스에서는 Pod가 기본 단위입니다.
- **서브넷 관점**: 클라우드 VPC에서 인스턴스는 서브넷 내에 배치되며 각각 서브넷 내 고유 IP를 가집니다. 쿠버네티스에서 각 Pod도 클러스터 내부 네트워크에서 고유한 IP를 할당받습니다.
- **네트워킹 모델**: AWS나 다른 클라우드 제공자의 VPC에서 동일한 서브넷의 인스턴스들이 직접 통신할 수 있는 것처럼, 쿠버네티스 클러스터 내의 Pod들도 서로 직접 통신할 수 있습니다.

## 일반적인 사용 패턴

- **싱글 컨테이너 Pod**: 가장 일반적인 사용 사례. 하나의 애플리케이션 컨테이너만 포함.
- **사이드카 패턴**: 주 애플리케이션 컨테이너와 함께 로깅, 모니터링, 프록시 등의 보조 기능을 수행하는 사이드카 컨테이너 포함.
- **앰배서더 패턴**: 네트워크 연결을 프록시하는 컨테이너를 포함.
- **어댑터 패턴**: 주 애플리케이션의 출력을 표준화하는 컨테이너 포함.

## 실습 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
      requests:
        memory: "64Mi"
        cpu: "250m"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 10
  restartPolicy: Always
```

## Pod 생명주기

- **Pending**: Pod가 쿠버네티스 시스템에 의해 수락되었지만 컨테이너가 아직 생성되지 않음
- **Running**: Pod가 노드에 바인딩되고 모든 컨테이너가 생성됨
- **Succeeded**: Pod의 모든 컨테이너가 성공적으로 종료되고 재시작되지 않음
- **Failed**: Pod의 모든 컨테이너가 종료되었고, 적어도 하나의 컨테이너가 실패로 종료됨
- **Unknown**: 어떤 이유로 Pod의 상태를 가져올 수 없음
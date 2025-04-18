---
{"dg-publish":true,"permalink":"/Kubernetes/▣ Deployment/"}
---


Deployment는 Pod와 ReplicaSet에 대한 선언적 업데이트를 제공하는 리소스입니다. 애플리케이션의 배포 상태를 관리하고 원하는 상태를 유지합니다.

## 특징

- Pod의 복제본 생성 및 관리
- 롤링 업데이트와 롤백 기능 제공
- 선언적 방식으로 애플리케이션 버전 관리
- 자동 복구 기능 (Pod 장애 발생 시 자동으로 새 Pod 생성)
- 스케일링 기능 (replicas 수를 조정하여 확장/축소 가능)
- 배포 기록 관리 및 이전 버전으로 복원 가능

## 클라우드 네트워크 개념과 비교

- **오토스케일링 그룹**: AWS의 Auto Scaling Group이나 Azure의 Virtual Machine Scale Sets처럼 Deployment는 Pod의 복제본을 관리하고 스케일링합니다.
- **블루/그린 배포**: 클라우드 환경에서의 블루/그린 배포 전략과 유사하게, Deployment는 롤링 업데이트를 통해 무중단 배포를 지원합니다.
- **버전 관리**: 클라우드의 이미지 버전 관리 또는 인프라 코드화(IaC) 도구와 유사하게, Deployment는 애플리케이션 버전을 선언적으로 관리합니다.

## 실습 예시

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
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
```

## Deployment 주요 기능

**롤링 업데이트: 애플리케이션을 점진적으로 업데이트하여 다운타임 없이 새 버전 배포**

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

**롤백: 이전 버전으로 되돌리기**

```bash
kubectl rollout undo deployment/nginx-deployment
```

**스케일링: Pod 개수 조정**

```bash
kubectl scale deployment/nginx-deployment --replicas=5
```

**배포 일시 중지/재개: 업데이트 과정 제어**

```bash
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
```

**배포 상태 확인:**

```bash
kubectl rollout status deployment/nginx-deployment
```

## Deployment와 관련된 리소스

- **ReplicaSet**: Deployment에 의해 생성되며 Pod 복제본 집합 관리
- **Pod**: 실제 애플리케이션 인스턴스를 실행
- **HorizontalPodAutoscaler**: CPU 사용률 등에 따라 Deployment의 replicas 수를 자동으로 조정
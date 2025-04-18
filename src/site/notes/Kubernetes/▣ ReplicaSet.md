---
{"dg-publish":true,"permalink":"/Kubernetes/▣ ReplicaSet/"}
---


ReplicaSet은 지정된 수의 Pod 복제본이 항상 실행되도록 보장하는 쿠버네티스 리소스입니다. Pod의 가용성을 유지하고 애플리케이션의 수평적 확장을 관리합니다.

## 특징

- 지정된 수의 Pod 복제본을 항상 유지
- Pod 장애 시 자동으로 새 Pod 생성
- 레이블 셀렉터를 통해 관리할 Pod 지정
- Deployment의 기반이 되는 리소스 (직접 사용보다는 Deployment 통해 사용 권장)
- Pod 템플릿을 사용하여 새 Pod 생성

## Deployment와의 관계

- Deployment는 ReplicaSet을 생성하고 관리
- Deployment는 ReplicaSet의 기능에 롤링 업데이트 및 롤백 기능 추가
- 일반적으로 ReplicaSet을 직접 사용하기보다 Deployment를 사용 권장

## 클라우드 네트워크 개념과 비교

- **오토스케일링 그룹**: AWS Auto Scaling Group, Azure VM Scale Sets과 유사하게 인스턴스 수 유지
- **로드 밸런서 백엔드 풀**: 부하 분산을 위한 서버 그룹과 유사한 개념

## ReplicaSet 예시

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
```

## 셀렉터 타입

ReplicaSet은 두 가지 타입의 셀렉터를 지원합니다:

**matchLabels**: 키-값 쌍과 정확히 일치하는 Pod 선택

```yaml
selector:
  matchLabels:
    tier: frontend
```

**matchExpressions**: 더 복잡한 셀렉터 표현식 지원

```yaml
selector:
  matchExpressions:
  - key: tier
    operator: In
    values:
    - frontend
    - backend
  - key: environment
    operator: NotIn
    values:
    - dev
```

## ReplicaSet 작동 방식

- 현재 실행 중인 Pod 수가 replicas 값보다 적으면 새 Pod 생성
- 현재 실행 중인 Pod 수가 replicas 값보다 많으면 초과 Pod 삭제
- Pod 장애가 발생하면 자동으로 새 Pod 생성하여 replicas 수 유지
- Pod 템플릿의 변경은 기존 Pod에 영향을 주지 않음 (Deployment와 달리 자동 업데이트 없음)

## 명령적 방식으로 ReplicaSet 관리

```bash
# ReplicaSet 생성
kubectl create -f replicaset.yaml

# ReplicaSet 목록 조회
kubectl get rs

# ReplicaSet 상세 정보 조회
kubectl describe rs/frontend

# ReplicaSet 스케일링
kubectl scale rs/frontend --replicas=5

# ReplicaSet 삭제 (Pod도 함께 삭제)
kubectl delete rs/frontend

# ReplicaSet만 삭제 (Pod는 유지)
kubectl delete rs/frontend --cascade=false
```
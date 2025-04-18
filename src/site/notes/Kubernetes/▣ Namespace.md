---
{"dg-publish":true,"permalink":"/kubernetes/namespace/"}
---

Namespace는 쿠버네티스 클러스터 내에서 리소스들을 논리적으로 분리하는 가상 환경입니다. 하나의 물리적 클러스터를 여러 가상 클러스터로 나누어 사용할 수 있게 해줍니다.

## 특징

- 리소스 이름의 범위 지정 (동일한 이름의 리소스가 다른 네임스페이스에 존재 가능)
- 다중 팀 또는 프로젝트 환경에서 리소스 분리
- 권한 분리를 위한 RBAC 적용의 기본 단위
- 리소스 할당량(Resource Quota) 적용 가능
- 네트워크 정책을 통한 네임스페이스 간 통신 제어 가능

## 기본 네임스페이스

- **default**: 별도로 네임스페이스를 지정하지 않으면 사용되는 기본 네임스페이스
- **kube-system**: 쿠버네티스 시스템 컴포넌트를 위한 네임스페이스
- **kube-public**: 모든 사용자(인증되지 않은 사용자 포함)가 읽을 수 있는 리소스를 위한 네임스페이스
- **kube-node-lease**: 노드 하트비트를 위한 네임스페이스

## 클라우드 네트워크 개념과 비교

- **계정 분리**: AWS 계정, Azure 구독, Google 프로젝트와 같이 리소스를 분리하는 방식과 유사
- **VPC/서브넷**: 클라우드의 VPC나 서브넷처럼 논리적 네트워크 분리와 유사한 개념
- **리소스 그룹**: Azure Resource Group이나 AWS Resource Groups처럼 관련 리소스를 그룹화하는 방식과 유사

## 네임스페이스 생성 예시

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    env: dev
```

## 특정 네임스페이스에 리소스 생성 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: development
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
```

## 네임스페이스 리소스 할당량 설정 예시

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "8"
    limits.memory: 10Gi
```

## 네임스페이스 네트워크 정책 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-from-other-namespaces
  namespace: development
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}  # 같은 네임스페이스의 Pod만 허용
```

## 네임스페이스 관련 명령어

```bash
# 네임스페이스 생성
kubectl create namespace development

# 네임스페이스 목록 조회
kubectl get namespaces

# 특정 네임스페이스의 리소스 조회
kubectl get pods -n development

# 현재 컨텍스트의 기본 네임스페이스 변경
kubectl config set-context --current --namespace=development

# 모든 네임스페이스의 리소스 조회
kubectl get pods --all-namespaces
```

## 네임스페이스 제한사항

- 클러스터 범위 리소스는 네임스페이스에 속하지 않음 (예: Node, PersistentVolume, Namespace 자체)
- 네임스페이스 간에는 DNS를 통한 서비스 액세스 가능 (<서비스>.<네임스페이스>.svc.cluster.local)
- 일부 리소스는 네임스페이스를 넘어 사용 가능 (예: StorageClass)

## 네임스페이스 사용 모범 사례

- 환경별 구분 (dev, test, staging, production)
- 팀 또는 프로젝트별 구분
- 각 네임스페이스에 적절한 리소스 할당량 설정
- RBAC를 통한 접근 제어 구현
- 명확한 네이밍 규칙 적용
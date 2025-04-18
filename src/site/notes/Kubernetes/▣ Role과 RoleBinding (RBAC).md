---
{"dg-publish":true,"permalink":"/Kubernetes/▣ Role과 RoleBinding (RBAC)/"}
---


Role-Based Access Control(RBAC)은 쿠버네티스 리소스에 대한 접근 권한을 관리합니다.

## 주요 구성 요소

- **Role**: 특정 네임스페이스 내 리소스에 대한 권한 정의
- **ClusterRole**: 클러스터 전체 리소스에 대한 권한 정의
- **RoleBinding**: 사용자와 Role 연결
- **ClusterRoleBinding**: 사용자와 ClusterRole 연결

## 클라우드 네트워크 개념과 비교

- **IAM 정책**: AWS IAM이나 Azure RBAC와 유사하게, 쿠버네티스 RBAC는 리소스에 대한 세부적인 접근 제어를 제공합니다.
- **최소 권한 원칙**: 클라우드 보안 모범 사례와 마찬가지로, 쿠버네티스 RBAC는 필요한 최소한의 권한만 부여하는 방식을 권장합니다.

## 실습 예시

### Role 정의 예시

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]      # 코어 API 그룹
  resources: ["pods"]  # 리소스 타입
  verbs: ["get", "watch", "list"]  # 허용되는 동작
```

### ClusterRole 정의 예시

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

### RoleBinding 정의 예시

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane  # 사용자 "jane"
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: default
  namespace: kube-system
roleRef:
  kind: Role
  name: pod-reader  # 위에서 정의한 Role
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRoleBinding 정의 예시

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager  # "manager" 그룹의 모든 사용자
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader  # 위에서 정의한 ClusterRole
  apiGroup: rbac.authorization.k8s.io
```

## RBAC 관리

### 권한 확인

```bash
kubectl auth can-i get pods --namespace dev --as jane
```

### Role 및 RoleBinding 확인

```bash
kubectl get roles
kubectl get rolebindings
kubectl get clusterroles
kubectl get clusterrolebindings
```

### 특정 Role 상세 정보 보기

```bash
kubectl describe role pod-reader
```

## 일반적인 사용 패턴

- **네임스페이스 관리자**: 특정 네임스페이스 내 모든 리소스 관리 권한
- **개발자 역할**: 애플리케이션 배포 및 로그 확인 권한
- **읽기 전용 사용자**: 모니터링 및 감사를 위한 읽기 전용 접근
- **CI/CD 파이프라인**: 배포 자동화를 위한 제한된 권한
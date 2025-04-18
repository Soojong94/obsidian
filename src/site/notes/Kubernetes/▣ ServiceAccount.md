---
{"dg-publish":true,"permalink":"/Kubernetes/▣ ServiceAccount/"}
---


ServiceAccount는 Pod가 쿠버네티스 API와 상호작용할 때 사용하는 ID입니다.

## 특징

- Pod의 쿠버네티스 API 접근 권한 관리
- 네임스페이스에 종속
- RBAC와 연계하여 세밀한 권한 제어 가능
- 기본적으로 각 Pod에 할당됨

## 클라우드 네트워크 개념과 비교

- **인스턴스 프로필**: AWS IAM 인스턴스 프로필이나 Azure 관리 ID와 유사하게, ServiceAccount는 워크로드에 ID를 제공합니다.
- **최소 권한**: 클라우드 보안 모범 사례처럼, 애플리케이션에 필요한 최소한의 권한만 부여하는 방식을 지원합니다.

## 실습 예시

### ServiceAccount 생성

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring-app
  namespace: monitoring
```

### Pod에 ServiceAccount 할당

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: metrics-collector
  namespace: monitoring
spec:
  serviceAccountName: monitoring-app
  containers:
  - name: collector
    image: metrics-collector:v1.2
```

### RBAC와 함께 사용 (RoleBinding)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metrics-viewer
  namespace: default
subjects:
- kind: ServiceAccount
  name: monitoring-app
  namespace: monitoring
roleRef:
  kind: Role
  name: pod-metrics-reader
  apiGroup: rbac.authorization.k8s.io
```

### 토큰 자동 마운트 비활성화

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: restricted-app
automountServiceAccountToken: false
```

## ServiceAccount 관리

### 계정 확인

```bash
kubectl get serviceaccounts
kubectl describe serviceaccount monitoring-app
```

### 토큰 확인

```bash
kubectl describe secret $(kubectl get secrets | grep monitoring-app | awk '{print $1}')
```

### Pod 내에서 ServiceAccount 토큰 사용

```bash
# Pod 내부에서 실행
curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

## 보안 고려사항

- 항상 최소 권한 원칙 적용
- 기본 ServiceAccount 권한 제한
- 필요할 때만 토큰 자동 마운트 활성화
- 정기적으로 ServiceAccount 권한 감사

## 일반적인 사용 패턴

- **CI/CD 파이프라인**: 배포 도구에 제한된 권한 부여
- **모니터링 시스템**: 클러스터 메트릭 수집용 특수 계정
- **운영자 패턴**: 커스텀 컨트롤러에 특정 리소스 관리 권한 부여
- **외부 서비스 연동**: 외부 서비스가 클러스터와 안전하게 상호작용하도록 지원
---
{"dg-publish":true,"permalink":"/kubernetes/network-policy/"}
---


NetworkPolicy는 Pod 간 네트워크 트래픽을 제어하는 방화벽 규칙과 같은 리소스입니다.

## 특징

- Pod 간 통신 제어
- 네임스페이스 간 통신 제어
- 인그레스(들어오는) 및 이그레스(나가는) 트래픽 정책 설정
- 마이크로서비스 보안에 중요

## 클라우드 네트워크 개념과 비교

- **보안 그룹**: AWS 보안 그룹이나 Azure NSG(Network Security Group)와 유사하게, NetworkPolicy는 트래픽 필터링을 제공합니다.
- **마이크로세그멘테이션**: 클라우드 보안 전략에서의 마이크로세그멘테이션처럼, NetworkPolicy는 세밀한 네트워크 격리를 가능하게 합니다.

## 실습 예시

### 기본 NetworkPolicy 정의 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
    ports:
    - protocol: TCP
      port: 80
```

### 네임스페이스 간 통신 정책 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: frontend
      podSelector:
        matchLabels:
          app: web
    ports:
    - protocol: TCP
      port: 8080
```

### 이그레스(나가는) 트래픽 제한 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: limit-outbound
  namespace: restricted
spec:
  podSelector:
    matchLabels:
      app: internal-app
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53  # DNS 요청 허용
```

## NetworkPolicy 관리

### 정책 확인

```bash
kubectl get networkpolicies
kubectl describe networkpolicy access-nginx
```

### 정책 테스트

```bash
# 임시 Pod 생성하여 연결 테스트
kubectl run test-pod --rm -it --image=alpine --labels="access=true" -- sh
```

## 구현 요구사항

NetworkPolicy는 CNI(Container Network Interface) 플러그인에 의해 구현됩니다. 모든 쿠버네티스 클러스터가 기본적으로 NetworkPolicy를 지원하는 것은 아닙니다. 다음과 같은 CNI 플러그인이 NetworkPolicy를 지원합니다:

- Calico
- Cilium
- Kube-router
- Weave Net
- Antrea

## 일반적인 사용 패턴

- **기본 거부 정책**: 명시적으로 허용되지 않은 모든 트래픽 차단
- **애플리케이션 계층 격리**: 프론트엔드, 백엔드, 데이터베이스 등 계층별 통신 제어
- **네임스페이스 격리**: 개발, 테스트, 프로덕션 환경 간 격리
- **규정 준수**: PCI-DSS, HIPAA 등 보안 규정 준수를 위한 네트워크 세분화
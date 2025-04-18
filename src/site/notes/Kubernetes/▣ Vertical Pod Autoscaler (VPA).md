---
{"dg-publish":true,"permalink":"/kubernetes/vertical-pod-autoscaler-vpa/"}
---



VPA는 Pod의 CPU 및 메모리 요청과 제한을 자동으로 조정하여 리소스 활용도를 최적화합니다. Pod 수를 변경하는 HPA와 달리, VPA는 각 Pod의 리소스 할당을 조정합니다.

## 특징

- CPU 및 메모리 요청을 자동으로 조정
- 여러 모드 지원: Auto, Recreate, Initial, Off
- 리소스 활용도 최적화 및 비용 절감
- VPA 컨트롤러 설치 필요

## 실습 예시

### Auto

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: analytics-vpa
  namespace: default
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: analytics-deployment
  updatePolicy:
    updateMode: "Auto"
```

## 주요 필드 설명

- **targetRef**: VPA가 관리할 대상 리소스
- **updatePolicy.updateMode**: VPA 동작 모드 설정
    - **Auto**: 자동으로 Pod를 재생성하여 리소스 요청 업데이트
    - **Recreate**: 리소스 요청 변경 시 Pod 재생성 (자동 축출)
    - **Initial**: 새 Pod에만 리소스 권장사항 적용
    - **Off**: 권장사항 계산만 수행 (적용 안함)
---
{"dg-publish":true,"permalink":"/kubernetes/horizontal-pod-autoscaler-hpa/"}
---


HPA는 CPU 사용률이나 기타 선택된 메트릭에 따라 워크로드 리소스(Deployment, StatefulSet 등)의 Pod 수를 자동으로 조정합니다.

## 특징

- 메트릭 기반 자동 스케일링 (일반적으로 CPU 사용률)
- 최소 및 최대 Pod 수 설정 가능
- 스케일 업/다운 동작 커스터마이징 가능
- metrics-server가 필요함

## 실습 예시

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kkapp-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
```

## 주요 필드 설명

- **scaleTargetRef**: 스케일링할 대상 리소스
- **minReplicas/maxReplicas**: 최소/최대 복제본 수
- **metrics**: 스케일링 기준 메트릭 (CPU, 메모리, 사용자 정의 메트릭 등)
- **behavior**: 스케일 업/다운 동작 정의
- **stabilizationWindowSeconds**: 급격한 변동을 방지하기 위한 안정화 기간
---
{"dg-publish":true,"permalink":"/kubernetes/node-affinity-anti-affinity/"}
---


Node Affinity와 Anti-Affinity는 특정 노드 특성에 따라 Pod가 어떤 노드에 배치될지를 제어하는 메커니즘입니다.

## Node Affinity

Pod가 특정 노드 집합에 배치되도록 유도하는 규칙입니다. 노드의 레이블을 기반으로 합니다.

## Node Anti-Affinity

Pod가 특정 노드 집합에 배치되지 않도록 하는 규칙입니다.

## Pod Affinity/Anti-Affinity

다른 Pod와의 관계에 따라 Pod 배치를 제어하는 규칙입니다.

## 클라우드 네트워크 개념과 비교

- **가용 영역 분산**: 클라우드의 가용 영역 분산 배치 전략과 유사
- **리소스 풀**: 특정 유형의 리소스 풀에 워크로드 지정하는 것과 유사

## 실습 예시

### Node Affinity 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - zone1
  containers:
  - name: nginx
    image: nginx
```

### Pod Anti-Affinity 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    app: web
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - web
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: webapp
    image: nginx
```

## Affinity 타입

- **requiredDuringSchedulingIgnoredDuringExecution**: 반드시 충족해야 하는 조건 (하드 요구사항)
- **preferredDuringSchedulingIgnoredDuringExecution**: 가능하면 충족하는 조건 (소프트 요구사항)
- **requiredDuringSchedulingRequiredDuringExecution**: 계획 중인 기능, 런타임에도 조건 만족 필요

## 일반적인 사용 사례

- **하드웨어 특성 활용**: 특정 하드웨어 기능이 있는 노드에 워크로드 배치
- **고가용성 보장**: 다른 가용 영역에 Pod 분산 배치
- **파티셔닝**: 워크로드 유형별로 노드 분리
- **성능 최적화**: 연관된 Pod를 같은 노드/지역에 배치하여 지연 시간 최소화
- **동일 워크로드 분산**: 같은 애플리케이션의 여러 인스턴스를 다른 노드에 분산하여 가용성 향상

## Taint/Toleration과의 차이점

- **Taint/Toleration**: 노드가 Pod를 거부할 수 있는 메커니즘 ("이 노드에 스케줄링되지 마세요")
- **Node Affinity**: Pod가 특정 노드를 선호하는 메커니즘 ("이 노드에 스케줄링해 주세요")

두 기능은 종종 함께 사용되어 더 정교한 Pod 배치 전략을 구현합니다.
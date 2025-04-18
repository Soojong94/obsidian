---
{"dg-publish":true,"permalink":"/Kubernetes/Taint와 Toleration/"}
---


Taint와 Toleration은 특정 노드에 어떤 Pod가 스케줄링될 수 있는지를 제어하는 메커니즘입니다.

## Taint(테인트)

노드에 설정하는 속성으로, 이 노드에 특정 Pod만 스케줄링되도록 제한합니다.

## Toleration(톨러레이션)

Pod에 설정하는 속성으로, 특정 Taint가 있는 노드에도 스케줄링될 수 있는 권한을 부여합니다.

## 작동 방식

- 노드에 Taint를 설정하면 기본적으로 모든 Pod는 해당 노드에 스케줄링될 수 없음
- 해당 Taint를 허용하는 Toleration이 있는 Pod만 스케줄링 가능
- Taint와 Toleration은 일종의 "열쇠와 자물쇠" 관계

## 클라우드 네트워크 개념과 비교

- **서버 분리**: 기존 인프라에서 특정 서버를 특수 목적용으로 분리하는 것과 유사
- **네트워크 분리**: DMZ와 같은 네트워크 영역 분리와 개념적으로 유사

## 실습 예시

### 노드에 Taint 설정

```bash
# 노드에 Taint 추가
kubectl taint nodes node1 app=database:NoSchedule

# 노드의 Taint 확인
kubectl describe node node1 | grep Taint
```

### Pod에 Toleration 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database
spec:
  containers:
  - name: mysql
    image: mysql:5.7
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
```

## Taint 효과(Effect) 유형

- **NoSchedule**: 톨러레이션이 없는 Pod는 스케줄링되지 않음
- **PreferNoSchedule**: 톨러레이션이 없는 Pod는 가능하면 스케줄링하지 않음(선호)
- **NoExecute**: 톨러레이션이 없는 Pod는 스케줄링되지 않으며, 기존에 실행 중인 Pod도 제거됨

## 일반적인 사용 사례

- **전용 노드**: 특정 워크로드를 위한 전용 노드 구성
- **마스터 노드 보호**: 컨트롤 플레인 노드에 일반 워크로드가 스케줄링되지 않도록 방지
- **특수 하드웨어 노드**: GPU 또는 SSD가 장착된 노드에 관련 워크로드만 배치
- **유지보수**: 노드 유지보수 시 새로운 Pod 스케줄링 방지
- **문제 있는 노드 격리**: 문제가 있는 노드에 새 Pod 배치 방지
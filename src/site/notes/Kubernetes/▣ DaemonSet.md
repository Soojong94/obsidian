---
{"dg-publish":true,"permalink":"/Kubernetes/▣ DaemonSet/"}
---


DaemonSet은 모든 노드(또는 선택된 노드)에 Pod의 복제본을 하나씩 실행하도록 보장하는 쿠버네티스 워크로드 리소스입니다. 클러스터에 노드가 추가되면 자동으로 해당 노드에 Pod가 추가되고, 노드가 제거되면 해당 Pod도 제거됩니다.

## 특징

- 모든 노드(또는 선택된 노드)에 하나의 Pod 배포
- 노드 추가 시 자동으로 Pod 배포
- 노드 제거 시 가비지 컬렉션으로 Pod 정리
- 노드 라벨을 통한 선택적 배포 지원
- 시스템 데몬 또는 에이전트를 배포하는 데 적합

## Deployment와의 차이점

- 실행 위치: Deployment는 임의의 노드에 배포되지만, DaemonSet은 모든/선택된 노드에 배포
- 복제본 수: Deployment는 replicas 숫자로 지정하지만, DaemonSet은 노드 수에 따라 결정됨
- 배포 목적: Deployment는 애플리케이션 배포, DaemonSet은 노드 수준의 기능 제공

## 클라우드 네트워크 개념과 비교

- **시스템 에이전트**: 클라우드 VM에 설치하는 모니터링 에이전트, 로깅 에이전트와 유사
- **노드 레벨 서비스**: 각 서버에 설치하는 보안 스캐너, 네트워크 프록시와 유사
- **호스트 데몬**: 전통적인 Linux/UNIX 시스템의 시스템 데몬과 유사한 개념

## DaemonSet 예시 (로깅 에이전트)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # 컨트롤 플레인 노드에도 배포할 수 있도록 설정
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 특정 노드에만 배포하기 (노드 선택기)

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disk: ssd
```

## DaemonSet 업데이트 전략

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
```

- **RollingUpdate**: Pod를 점진적으로 업데이트 (기본값)
- **OnDelete**: Pod가 삭제될 때만 업데이트

## 일반적인 사용 사례

- **클러스터 스토리지 데몬**: Ceph, GlusterFS 등
- **로그 수집기**: Fluentd, Logstash
- **모니터링 에이전트**: Prometheus Node Exporter, Datadog Agent
- **네트워크 플러그인**: Calico, Weave Net
- **보안 에이전트**: Falco, AppArmor

## 명령적 방식 관리

```bash
# DaemonSet 생성
kubectl create -f daemonset.yaml

# DaemonSet 목록 조회
kubectl get daemonsets -n kube-system

# DaemonSet의 Pod 조회
kubectl get pods -l name=fluentd-elasticsearch -n kube-system

# DaemonSet 업데이트
kubectl apply -f updated-daemonset.yaml

# DaemonSet 삭제
kubectl delete daemonset fluentd-elasticsearch -n kube-system
```
---
{"dg-publish":true,"permalink":"/Kubernetes/▣ Static Pod/"}
---


Static Pod는 kubelet에 의해 직접 관리되는 Pod로, API 서버를 통하지 않고 특정 노드의 kubelet에 의해 직접 실행됩니다. 이러한 Pod는 /etc/kubernetes/manifests/ 디렉토리(또는 kubelet 구성에 지정된 다른 디렉토리)에 YAML 파일을 배치함으로써 생성됩니다.

## 특징

- API 서버 없이도 동작 가능
- 노드가 재시작되어도 자동으로 재시작됨
- 컨트롤 플레인 컴포넌트(etcd, kube-apiserver, kube-controller-manager, kube-scheduler)는 보통 Static Pod로 실행됨
- Static Pod 이름에는 자동으로 노드 이름이 접미사로 추가됨 (예: static-busybox-controlplane)

## 실습 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-busybox
  labels:
    run: static-busybox
spec:
  containers:
  - name: static-busybox
    image: busybox
    command:
    - sleep
    - "1000"
  restartPolicy: Always
```

## 오류 및 수정

실습 중 발생한 문제:

```
sleep: invalid number '/etc/kubernetes/manifests/'
```

원인: command 부분에 /etc/kubernetes/manifests/가 잘못 추가됨 해결: command 부분에서 불필요한 경로 인자 제거
---
{"dg-publish":true,"permalink":"/Kubernetes/NodePort 서비스 생성/"}
---


NodePort는 Kubernetes 서비스 타입 중 하나로, 클러스터 외부에서 내부 서비스에 접근할 수 있게 해줍니다. 각 클러스터 노드의 특정 포트를 개방하여 해당 포트로 들어오는 트래픽을 서비스로 라우팅합니다.

## 특징

- 노드 IP와 NodePort를 통해 서비스에 접근 가능
- NodePort 범위: 기본적으로 30000-32767
- 각 노드는 동일한 포트를 통해 서비스에 접근 가능
- 서비스 selector는 트래픽을 전달할 Pod를 결정

## 서비스 컴포넌트

- **Service Port**: 서비스 자체의 포트
- **Target Port**: 컨테이너가 리스닝하는 포트
- **Node Port**: 외부에서 접근 가능한 노드의 포트

## 실습 예시 - 선언적(Declarative) 방식

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hr-web-app-service
spec:
  selector:
    app: hr-web-app
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30082
  type: NodePort
```

## 실습 예시 - 명령적(Imperative) 방식

```bash
# 명령어만으로 서비스 생성
kubectl create service nodeport hr-web-app-service --tcp=8080:8080 --node-port=30082 --selector=app=hr-web-app

# 또는 기존 리소스를 노출
kubectl expose deployment hr-web-app --name=hr-web-app-service --port=8080 --target-port=8080 --type=NodePort --node-port=30082
```
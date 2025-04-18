---
{"dg-publish":true,"permalink":"/Kubernetes/▣ Service/"}
---

Service는 쿠버네티스에서 Pod 집합에 대한 네트워크 접근 방식을 정의하는 리소스입니다. Pod는 동적으로 생성되고 삭제되므로 IP 주소가 계속 변경되는데, Service는 이러한 Pod 집합에 대한 안정적인 네트워크 엔드포인트를 제공합니다.

## 특징

- 고정된 IP 주소와 DNS 이름 제공
- 로드 밸런싱 기능
- 서비스 디스커버리 메커니즘
- 내부(ClusterIP) 또는 외부(NodePort, LoadBalancer) 네트워크 노출 옵션
- 레이블 셀렉터를 사용해 대상 Pod 지정
- 세션 어피니티(Affinity) 지원

## 서비스 유형

- **ClusterIP (기본값)**: 클러스터 내부에서만 접근 가능한 IP 제공
- **NodePort**: 모든 노드의 특정 포트를 통해 서비스 노출 (포트 범위: 30000-32767)
- **LoadBalancer**: 클라우드 공급자의 로드 밸런서를 프로비저닝하여 서비스 노출
- **ExternalName**: 외부 서비스에 대한 DNS 별칭 제공

## 클라우드 네트워크 개념과 비교

- **로드 밸런서**: AWS ELB/ALB, Google Cloud Load Balancer와 유사한 역할을 하며 트래픽을 여러 Pod에 분산
- **DNS 라우팅**: Route 53이나 Cloud DNS와 같은 서비스처럼 서비스 디스커버리 제공
- **프록시/게이트웨이**: API Gateway나 Application Proxy와 같이 트래픽을 라우팅하고 관리
- **VPC 엔드포인트**: Private Link나 VPC Endpoint처럼 내부 서비스에 대한 접근점 제공

## 실습 예시

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: ClusterIP
  ports:
  - port: 80        # 서비스가 노출하는 포트
    targetPort: 80  # 대상 Pod의 포트
    protocol: TCP
  selector:
    app: nginx      # 이 레이블을 가진 Pod들이 서비스의 대상
```

## NodePort 서비스 예시

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # 노드에서 외부로 노출될 포트
  selector:
    app: nginx
```

## LoadBalancer 서비스 예시

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

## 서비스 디스커버리

- **환경 변수**: 서비스가 생성되면 Pod 내에서 환경 변수로 접근 가능
- **DNS**: 클러스터 내에서 <서비스이름>.<네임스페이스>.svc.cluster.local 형식의 DNS 이름으로 접근 가능

## 헤드리스 서비스

clusterIP: None으로 설정하여 단일 서비스 IP 없이 각 Pod의 DNS 레코드를 직접 제공하는 특수한 유형의 서비스입니다. StatefulSet과 함께 자주 사용됩니다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None
  selector:
    app: stateful-app
  ports:
  - port: 80
    targetPort: 80
```
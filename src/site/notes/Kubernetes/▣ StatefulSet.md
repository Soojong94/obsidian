---
{"dg-publish":true,"permalink":"/Kubernetes/▣ StatefulSet/"}
---


StatefulSet은 상태를 유지하는(stateful) 애플리케이션을 관리하기 위한 워크로드 리소스입니다. Deployment와 달리 Pod에 안정적인 네트워크 식별자와 영구 스토리지를 제공합니다.

## 특징

- 안정적이고 고유한 네트워크 식별자 제공
- 안정적이고 지속적인 스토리지 제공
- 순차적이고 정상적인 배포 및 스케일링
- 순차적이고 자동화된 롤링 업데이트
- Pod 순서 인덱스 유지 (0, 1, 2, ...)

## Deployment와의 차이점

- Pod 이름: <statefulset-name>-<index> 형식 사용 (예: mysql-0, mysql-1)
- 순차적 배포/확장/삭제: 인덱스 역순으로 처리 (n-1, n-2, ..., 0)
- 개별 PVC: 각 Pod마다 독립된 PVC 생성
- 헤드리스 서비스: Pod의 안정적인 네트워크 ID 제공

## 클라우드 네트워크 개념과 비교

- **데이터베이스 클러스터**: AWS RDS 멀티 AZ, Google Cloud Spanner, Azure Cosmos DB와 같은 분산 데이터베이스 시스템과 유사
- **메시지 큐 클러스터**: 분산 메시지 브로커와 유사한 아키텍처
- **분산 파일 시스템**: 여러 노드에 걸쳐 있는 분산 스토리지 시스템과 유사

## StatefulSet 예시 (MySQL)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: root-password
        ports:
        - name: mysql
          containerPort: 3306
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi
```

## 헤드리스 서비스 예시

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None  # 헤드리스 서비스
  selector:
    app: mysql
```

## Pod 식별자

StatefulSet의 각 Pod는 다음과 같은 형태의 DNS 이름을 가집니다:

```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

예: mysql-0.mysql.default.svc.cluster.local

## 일반적인 사용 사례

- 데이터베이스: MySQL, PostgreSQL, MongoDB 클러스터
- 메시지 큐: Kafka, RabbitMQ 클러스터
- 분산 캐시: Redis, Memcached 클러스터
- 분산 파일 시스템: Elasticsearch, Cassandra

## 배포 및 스케일링 동작

- 순차적 배포: 0, 1, 2, ... 순서로 배포
- 순차적 삭제: n-1, n-2, ..., 0 순서로 삭제
- 이전 Pod가 Running 및 Ready 상태가 되어야 다음 Pod 배포
- 각 Pod는 고유한 PVC를 가짐

## 업데이트 전략

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2  # 인덱스 2 이상의 Pod만 업데이트
```

- **RollingUpdate**: 역순으로 Pod를 하나씩 업데이트 (기본값)
- **OnDelete**: 수동으로 Pod를 삭제할 때만 업데이트
- **Partition**: 지정된 인덱스 이상의 Pod만 업데이트
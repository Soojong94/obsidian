---
{"dg-publish":true,"permalink":"/kubernetes/crd/"}
---



커스텀 리소스 정의(CRD)는 쿠버네티스 API를 확장하여 사용자 정의 리소스를 생성할 수 있게 해주며, 오퍼레이터는 이러한 커스텀 리소스를 관리하는 컨트롤러입니다.

## 커스텀 리소스 정의(CRD)

쿠버네티스 API의 확장을 통해 새로운 유형의 리소스를 정의할 수 있는 메커니즘입니다.

## 오퍼레이터(Operator)

커스텀 리소스의 상태를 모니터링하고 관리하는 컨트롤러로, 애플리케이션별 운영 지식을 자동화합니다.

## 클라우드 네트워크 개념과 비교

- **Infrastructure as Code**: 클라우드 템플릿(CloudFormation, ARM 템플릿 등)과 유사하게 복잡한 응용 프로그램을 선언적으로 배포/관리
- **서비스 오케스트레이션**: 클라우드 서비스 오케스트레이션 도구와 유사한 역할 수행

## 실습 예시

### 커스텀 리소스 정의(CRD) 생성

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    plural: databases
    singular: database
    shortNames:
    - db
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
              version:
                type: string
              storageSize:
                type: string
              replicas:
                type: integer
                minimum: 1
            required:
            - engine
            - version
            - storageSize
```

### 커스텀 리소스(CR) 생성

```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-postgres
spec:
  engine: postgres
  version: "13.3"
  storageSize: "10Gi"
  replicas: 3
```

## 오퍼레이터 구성 요소

- **커스텀 리소스 정의(CRD)**: API 확장을 위한 명세
- **컨트롤러**: 커스텀 리소스를 관찰하고 관리하는 로직
- **RBAC 권한**: 오퍼레이터가 필요한 리소스에 접근할 수 있는 권한

## 일반적인 사용 사례

- **데이터베이스 관리**: 데이터베이스 프로비저닝, 백업, 복구, 스케일링 자동화
- **메시징 시스템**: Kafka, RabbitMQ 등의 복잡한 메시징 시스템 관리
- **모니터링 시스템**: Prometheus, Elastic Stack 등의 배포 및 구성 자동화
- **서비스 메시**: Istio, Linkerd 등의 서비스 메시 관리
- **머신러닝 워크플로우**: 훈련 작업, 모델 서빙 등 ML 파이프라인 오케스트레이션

## 잘 알려진 오퍼레이터

- **Prometheus Operator**: 모니터링 스택 관리
- **Elasticsearch Operator**: Elasticsearch 클러스터 관리
- **etcd Operator**: etcd 클러스터 배포 및 관리
- **PostgreSQL Operator**: PostgreSQL 데이터베이스 관리
- **Istio Operator**: 서비스 메시 관리
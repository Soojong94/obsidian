---
{"dg-publish":true,"permalink":"/tips/g-rpc-bff/"}
---


## gRPC 개요

gRPC는 Google에서 개발한 고성능 오픈소스 RPC(Remote Procedure Call) 프레임워크입니다.

- **Protocol Buffers(protobuf)**: 언어 중립적인 효율적인 직렬화 도구로 데이터 구조 정의
- **HTTP/2 기반**: 연결 멀티플렉싱, 헤더 압축, 양방향 스트리밍 제공
- **다국어 지원**: Java, Go, C++, Python, Node.js 등 다양한 언어 지원
- **높은 성능**: JSON/XML보다 빠른 바이너리 직렬화, 적은 네트워크 사용량
- **강력한 타입 체크**: 컴파일 타임에 타입 검증으로 런타임 오류 감소

## BFF(Backend For Frontend) 패턴

BFF는 특정 프론트엔드 애플리케이션 유형에 최적화된 백엔드 서비스를 구축하는 아키텍처 패턴입니다.

- **목적별 API 집계**: 각 클라이언트 유형(웹, 모바일, 데스크톱)에 최적화된 API 제공
- **관심사 분리**: 프론트엔드 요구사항에 맞게 백엔드 로직 분리
- **응답 최적화**: 단일 요청으로 여러 마이크로서비스의 데이터 조합 가능
- **클라이언트 특화 캐싱**: 특정 프론트엔드에 필요한 데이터만 캐싱

## BFF에서의 gRPC 활용

BFF 패턴과 gRPC를 결합하면 다음과 같은 이점이 있습니다:

### 통신 흐름

```
프론트엔드 클라이언트 (HTTP/JSON) → BFF 계층 (HTTP → gRPC 변환) → 마이크로서비스 (gRPC 처리)
```

### 주요 장점

- **성능 최적화:**
    - 효율적인 바이너리 직렬화로 네트워크 부하 감소
    - 지연 시간 감소 및 처리량 증가
- **개발 효율성:**
    - 서비스 인터페이스를 .proto 파일로 명확하게 정의
    - 코드 생성으로 클라이언트/서버 구현 간소화
- **확장성:**
    - 서비스 메서드 추가가 용이함
    - 기존 인터페이스 유지하며 새 기능 추가 가능
- **다양한 통신 패턴:**
    - 단순 요청/응답 (Unary RPC)
    - 서버 스트리밍 RPC
    - 클라이언트 스트리밍 RPC
    - 양방향 스트리밍 RPC

## BFF에서 gRPC 구현 방식

### 클라이언트 구현 유형

- **블로킹(동기식) 스텁:**
    - 요청 후 응답을 받을 때까지 스레드 블로킹
    - 구현이 단순하지만 리소스 효율성 낮음
    - 예: ServiceNameGrpc.ServiceNameBlockingStub
- **비동기 스텁:**
    - 콜백 기반 비동기 호출 제공
    - 응답 대기 중 스레드 차단하지 않음
    - 예: ServiceNameGrpc.ServiceNameStub
- **리액티브 스텁 (gRPC-Java에서 지원):**
    - RxJava, Project Reactor 등과 통합
    - 스트림 기반 비동기 프로그래밍 모델
    - 예: ReactorServiceNameGrpc.reactorStub()

### 타임아웃 및 오류 처리

효과적인 gRPC 구현을 위한 중요 설정:

```java
// 타임아웃 설정 예시
ServiceNameGrpc.ServiceNameBlockingStub stub = ServiceNameGrpc.newBlockingStub(channel)
    .withDeadlineAfter(5, TimeUnit.SECONDS);

// 재시도 정책 설정
ManagedChannel channel = ManagedChannelBuilder.forAddress(host, port)
    .enableRetry()
    .maxRetryAttempts(3)
    .build();
```

### 서킷 브레이커 패턴 통합

서비스 장애 전파 방지를 위한 구현:

```java
// Resilience4j 사용 예시
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("serviceName");
Supplier<Response> decoratedSupplier = CircuitBreaker
    .decorateSupplier(circuitBreaker, () -> grpcStub.callService(request));
```

## BFF의 gRPC 최적화 전략

- **리소스 풀링**: gRPC 채널 및 연결 재사용
- **비동기 처리**: 블로킹 호출 대신 비동기/리액티브 접근법 활용
- **스마트 타임아웃**: 서비스별 적절한 타임아웃 설정
- **회로 차단기**: 장애 서비스 격리로 시스템 안정성 강화
- **적절한 로드 밸런싱**: 클라이언트 측 로드 밸런싱 구현
---
{"dg-publish":true,"permalink":"/trouble-shooting/spring-boot/"}
---



# gRPC 스레드 블로킹 문제 분석 및 해결 방안

## 문제 개요

- 프론트엔드 → BFF(Backend For Frontend) → 백엔드 마이크로서비스 구조에서 스레드 블로킹 발생
- BFF에서 백엔드 마이크로서비스로의 gRPC 호출 과정에서 대기 상태 지속
- 리소스 고갈 및 서비스 지연 초래

## 시스템 gRPC 구현 방식

- **UserGrpcClient 클래스**: 다양한 사용자 관련 백엔드 서비스와의 gRPC 통신 처리
- **블로킹 방식 호출**: `InfoUserServiceGrpc$InfoUserServiceBlockingStub`과 같은 블로킹 스텁 사용
- **Protocol Buffer 정의**: .proto 파일에 정의된 서비스 인터페이스
- **현재 이슈**: gRPC 블로킹 호출에 타임아웃 설정이 없어 백엔드 서비스 장애 시 BFF 스레드가 무한정 대기

### GRPC란? [[Tips/gRPC 통신 및 BFF 아키텍처\|gRPC 통신 및 BFF 아키텍처]]

## 문제 패턴 요약

모든 문제는 프론트엔드에서 BFF로의 HTTP 요청으로 시작되어, BFF에서 backend-service-user-prd 서비스로의 gRPC 호출에서 응답을 받지 못하고 무한정 대기하는 상황. 대부분의 병목은 UserGrpcClient 클래스의 메서드에서 발생하며, 주로 사용자 정보, 좋아요 정보, 그룹 정보 등을 요청할 때 발생.

![VpXeqkKFJvHojydZM8uHpOzqrmIhLOr3GXN7JGS0jwc=_.png](/img/user/VpXeqkKFJvHojydZM8uHpOzqrmIhLOr3GXN7JGS0jwc=_.png)
## 스레드 덤프 분석 결과

- 다수의 HTTP 요청 처리 스레드가 WAITING 상태로 차단됨
- 차단 지점: `io.grpc.stub.ClientCalls$ThreadlessExecutor.waitAndDrain`
- 일부 스레드는 79분 이상 차단된 상태로 유지
- 대부분 gRPC 블로킹 호출에서 발생

```
"http-nio-8091-exec-116" #20438 [20488] daemon prio=5 os_prio=0 cpu=25.55ms elapsed=4792.99s tid=0x00007f069c027780 nid=20488 waiting on condition  [0x00007f064e6fc000]
   java.lang.Thread.State: WAITING (parking)
        at jdk.internal.misc.Unsafe.park(java.base@21.0.6/Native Method)
        - parking to wait for  <0x00000000b2f26d88> (a io.grpc.stub.ClientCalls$ThreadlessExecutor)
        at java.util.concurrent.locks.LockSupport.park(java.base@21.0.6/LockSupport.java:221)
        at io.grpc.stub.ClientCalls$ThreadlessExecutor.waitAndDrain(ClientCalls.java:717)
        at io.grpc.stub.ClientCalls.blockingUnaryCall(ClientCalls.java:159)
        at kr.jnmall.user.InfoUserServiceGrpc$InfoUserServiceBlockingStub.findInfoUserByAccountId(InfoUserServiceGrpc.java:1686)
        at kr.jnmall.inf.service.UserGrpcClient.findInfoUserByAccountId(UserGrpcClient.java:254)
        at kr.jnmall.user.service.UserAuthGrpcClientService.partnerLogin(UserAuthGrpcClientService.java:61)
        at kr.jnmall.user.controller.UserAuthController.login(UserAuthController.java:75)
```

## 문제가 발생하는 주요 흐름

### 1. 로그인 요청 흐름

```
클라이언트 → HTTP POST 요청 → UserAuthController.login() → UserAuthGrpcClientService.partnerLogin() 
→ UserGrpcClient.findInfoUserSearch() → InfoUserServiceGrpc.findInfoUserSearch() (gRPC 호출) 
→ backend-service-user-prd (여기서 응답하지 않음)
```

스레드 덤프에서 확인된 구체적인 경로:

```
jakarta.servlet.http.HttpServlet.service() 
→ FrameworkServlet.doPost() 
→ UserAuthController.login() 
→ UserAuthGrpcClientService.partnerLogin() 
→ UserGrpcClient.findInfoUserSearch() 
→ blockingUnaryCall() (여기서 대기 중)
```

### 2. 상품 목록 요청 흐름

```
클라이언트 → HTTP GET 요청 → ProductController.findInfoThemeProductList()/findInfoPlanProductList() 
→ ProductGrpcClientService.findInfoProductList() 
→ UserGrpcClient.findInfoUserLikes() → InfoUserLikesServiceGrpc.findInfoUserLikes() (gRPC 호출) 
→ backend-service-user-prd (여기서 응답하지 않음)
```

### 3. 메인 페이지 배너 요청 흐름

```
클라이언트 → HTTP GET 요청 → MainPageController.findInfoBannerPagination() 
→ MainPageGrpcClientService.findInfoBannerPagination() 
→ MainPageGrpcClientService.getUserGroups() 
→ UserGrpcClient.findInfoUserGroup() → InfoUserGroupServiceGrpc.findInfoUserGroup() (gRPC 호출) 
→ backend-service-user-prd (여기서 응답하지 않음)
```

## 백엔드 서비스 문제 증거

```
2025-03-19T13:48:03 ERROR : INTERNAL: Unexpected error: java.lang.OutOfMemoryError: Java heap space
io.grpc.StatusRuntimeException: INTERNAL: Unexpected error: java.lang.OutOfMemoryError: Java heap space
        at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:268)

2025-03-19T13:48:03 ERROR : INTERNAL: Unexpected error: Could not open JPA EntityManager for transaction
io.grpc.StatusRuntimeException: INTERNAL: Unexpected error: Could not open JPA EntityManager for transaction
        at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:268)

2025-03-19T13:56:21 ERROR : INTERNAL: Unexpected error: Hibernate transaction: Unable to rollback against JDBC Connection; Communications link failure during rollback(). Transaction resolution unknown.
```

## 근본 원인 분석

### 백엔드 서비스 안정성 문제

- backend-service-user-prd에서 메모리 부족 및 DB 연결 이슈
- 일부 요청에 대한 타임아웃 없이 무한정 대기

### gRPC 통신 구성 문제

- 타임아웃 설정 부재
- 적절한 예외 처리 부족
- 대기 중인 연결이 스레드 풀 고갈 초래

### 리소스 관리 이슈

- 스레드 풀 고갈로 신규 요청 처리 불가
- 장기간 연결 유지로 메모리 사용 증가

## 해결 전략

### 1. 타임아웃 설정

```java
// gRPC 채널 생성 및 데드라인 설정
ManagedChannel channel = ManagedChannelBuilder.forAddress("backend-service", 9090)
    .usePlaintext()
    .build();
   
// 블로킹 스텁에 3초 데드라인 설정
BackendServiceGrpc.BackendServiceBlockingStub stub = BackendServiceGrpc.newBlockingStub(channel)
    .withDeadlineAfter(3, TimeUnit.SECONDS);
```

### 2. 서킷 브레이커 패턴 적용

```java
@CircuitBreaker(name = "backendService", fallbackMethod = "fallbackMethod")
public Response callBackendService(Request request) {
    // 백엔드 서비스 호출 로직
    return backendStub.processRequest(request);
}

// 폴백 메서드
public Response fallbackMethod(Request request, Exception e) {
    log.error("Circuit breaker 활성화: {}", e.getMessage());
    return Response.newBuilder()
            .setStatus("SERVICE_UNAVAILABLE")
            .setMessage("현재 서비스 이용량이 많습니다. 잠시 후 다시 시도해주세요.")
            .build();
}
```

### 3. 백프레셔 메커니즘 구현

```java
// 세마포어를 이용한 동시 요청 제한
private final Semaphore semaphore = new Semaphore(10);

public Response callBackendServiceWithBackpressure(Request request) {
    try {
        if (!semaphore.tryAcquire(500, TimeUnit.MILLISECONDS)) {
            // 요청이 너무 많은 경우 즉시 반환
            return createTooManyRequestsResponse();
        }
        
        try {
            return backendStub.processRequest(request);
        } finally {
            semaphore.release();
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new RuntimeException("요청이 중단되었습니다", e);
    }
}
```

### 4. 통합 구현 예시

```java
@Service
public class ResilientBackendService {
   
    @CircuitBreaker(name = "backendService", fallbackMethod = "fallbackMethod")
    @Bulkhead(name = "backendService", fallbackMethod = "fallbackMethod") 
    @TimeLimiter(name = "backendService", fallbackMethod = "timeoutFallback")
    public CompletableFuture<Response> callBackendService(Request request) {
        return CompletableFuture.supplyAsync(() -> {
            return backendStub
                    .withDeadlineAfter(3, TimeUnit.SECONDS)
                    .processRequest(request);
        });
    }
    
    // 폴백 메서드들...
}
```

## 주의사항

- 이 전략들은 장애 격리 목적의 임시 방편임
- 근본적인 문제(slow query 등)는 별도 해결 필요

## 목적

- 시스템 전체의 장애 전파 방지
- 일부 사용자에게라도 서비스 계속 제공
- 서버 리소스 완전 고갈 방지

## 추가 권장사항

- 백엔드 서비스의 DB 쿼리 최적화
- DB 인덱싱 개선
- 비동기 처리 방식 도입 검토
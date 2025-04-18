---
{"dg-publish":true,"permalink":"/Tips/Supabase 개요/"}
---

# Supabase 기능 가이드

## Part 1: 기본 구조 및 클라이언트 SDK

**Firebase의 대안**

- 오픈소스로 제공되어 자체 호스팅이 가능
- PostgreSQL을 기반
- Supabase CLI를 통해 데이터베이스 구조를 자동으로 분석하고 타입을 생성

### 1. 기본 아키텍처 구조

아키텍처 3계층 구성:

- 클라이언트 계층 (Client Layer)
- 서비스 계층 (Service Layer)
- 데이터베이스 계층 (Database Layer)

![Pasted image 20250418131729.png](/img/user/images/Pasted%20image%2020250418131729.png)

### Client Application

**주요 특징**

- 모던 프레임워크 지원
    - 다양한 프레임워크용 전용 클라이언트 라이브러리 존재
    - React용 hooks, Vue용 composables 등 프레임워크별 최적화 도구
    - 각 프레임워크의 상태 관리 시스템과의 원활한 통합
- 타입스크립트 지원
    - 데이터베이스 스키마 기반의 자동 타입 생성
    - IDE 자동완성 기능 활용 극대화
    - 런타임 이전 타입 관련 오류 사전 포착
- 자동 생성된 타입 정의
    - CLI 도구를 통한 스키마 기반 자동 타입 생성
    - 스키마 변경 시 타입 자동 업데이트와 동기화
    - 복잡한 조인/관계 쿼리의 정확한 타입 추론

### SDK

**주요 특징**

- 데이터베이스 스키마 기반 타입 생성
    - 데이터베이스 구조의 자동 분석과 타입스크립트 정의 파일 생성
    - 데이터 모델의 완벽한 타입 안정성과 자동완성 지원
    - 복잡한 관계형 데이터 모델의 자동 타입 매핑
- 런타임 에러 방지 메커니즘
    - 요청 전송 전 데이터 형식과 유효성의 사전 검증
    - SQL 인젝션 방지를 위한 자동 쿼리 파라미터화
    - 네트워크 오류와 타임아웃에 대한 자동 복구 시스템
- 실시간 데이터 구독 기능
    - WebSocket 기반의 즉각적인 데이터 변경 감지 시스템
    - 테이블 또는 개별 레코드 단위의 세분화된 구독
    - 이벤트 타입별 선택적 구독과 필터링 기능
- 자동화된 API 요청 처리
    - RESTful 엔드포인트의 자동 생성과 라우팅(내부적으로 fetch API)
    - 단순 메소드 호출을 통한 복잡한 API 통신 구현
    - 요청 재시도, 취소, 타임아웃의 자동 관리 기능
- 인증 상태 관리 통합
    - JWT 기반 인증과 자동 토큰 갱신 메커니즘
    - 다양한 인증 방식의 통합 인터페이스 제공
    - 보안 토큰의 안전한 저장과 관리 시스템
- 파일 스토리지 작업 단순화
    - 대용량 파일의 효율적인 업로드/다운로드 처리
    - 청크 단위 전송과 진행률 모니터링 기능
    - CDN 통합을 통한 글로벌 콘텐츠 전송 최적화

### Supabase Client SDK

```javascript
import { createClient } from '@supabase/supabase-js'

// SDK 초기화
const supabase = createClient(
  'https://your-project.supabase.co',
  'your-anon-key'
)

// 기본 CRUD 작업 유틸리티
const DataService = {
  // 데이터 생성
  async createRecord(table: string, data: any) {
    const { data: result, error } = await supabase
      .from(table)
      .insert(data)
      .select()
    return { result, error }
  },

  // 데이터 조회
  async getRecords(table: string) {
    const { data, error } = await supabase
      .from(table)
      .select('*')
    return { data, error }
  },

  // 데이터 수정
  async updateRecord(table: string, id: number, data: any) {
    const { data: result, error } = await supabase
      .from(table)
      .update(data)
      .eq('id', id)
      .select()
    return { result, error }
  },

  // 데이터 삭제
  async deleteRecord(table: string, id: number) {
    const { error } = await supabase
      .from(table)
      .delete()
      .eq('id', id)
    return { error }
  }
}
```

## Part 2: 서비스 계층

### 1. Auth (GoTrue)

**주요 특징**

- JWT 기반 인증 시스템
    - 산업 표준 JWT를 활용한 확장 가능한 토큰 관리
    - 사용자 권한과 메타데이터의 토큰 내 안전한 저장
    - 토큰 수명 주기와 자동 갱신 메커니즘 내장
- 소셜 로그인 통합
    - Google, GitHub, Facebook 등 주요 OAuth 제공자 즉시 연동
    - 통일된 인터페이스를 통한 다중 인증 제공자 관리
    - 소셜 계정 연동과 프로필 데이터 동기화 기능
- 이메일/비밀번호 인증
    - 안전한 패스워드 해싱과 솔팅 자동 처리
    - 이메일 인증과 비밀번호 재설정 워크플로우
    - 사용자 계정 잠금과 보안 정책 설정 기능
- 매직 링크 인증
    - 일회용 인증 링크를 통한 비밀번호 없는 로그인
    - 커스텀 이메일 템플릿과 브랜딩 요소 적용 가능
    - 링크 만료 시간과 사용 횟수 제한 설정
- 세션 관리 기능
    - 다중 디바이스 세션의 동시 관리와 추적
    - 세션 만료와 강제 로그아웃 메커니즘
    - 디바이스별 접근 기록과 활동 로깅
- 사용자 메타데이터 관리
    - 커스텀 사용자 프로필 데이터의 유연한 저장
    - 역할 기반 접근 제어를 위한 권한 관리
    - 사용자 데이터의 자동 백업과 복구 시스템

**구현 예시**

```javascript
const AuthService = {
  // 이메일 회원가입
  async signUp(email: string, password: string) {
    const { data, error } = await supabase.auth.signUp({
      email,
      password
    })
    return { data, error }
  },

  // 이메일 로그인
  async signIn(email: string, password: string) {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    })
    return { data, error }
  },

  // OAuth 로그인
  async signInWithProvider(provider: 'google' | 'github' | 'facebook') {
    const { data, error } = await supabase.auth.signInWithOAuth({
      provider
    })
    return { data, error }
  }
}
```

### 2. Storage

**주요 특징**

- S3 호환 스토리지 시스템
    - AWS S3와 완벽한 호환성을 갖춘 API 구조
    - 대용량 파일의 효율적 업로드와 다운로드 처리
    - 버킷 기반의 체계적인 파일 조직화 시스템
- CDN 자동 통합
    - 글로벌 엣지 로케이션을 통한 콘텐츠 전송
    - 자동 캐싱과 무효화를 통한 성능 최적화
    - 지역별 접근 속도 최적화와 트래픽 분산
- 파일 메타데이터 관리
    - 파일별 커스텀 메타데이터 설정과 검색
    - 자동화된 파일 버전 관리와 이력 추적
    - MIME 타입과 파일 속성의 자동 감지
- 접근 권한 제어
    - 파일과 폴더 단위의 세분화된 접근 제어
    - 임시 URL 생성을 통한 제한적 접근 제공
    - RLS와 통합된 동적 권한 관리 시스템
    - Database-level Authentication / Serverless Authentication
- 이미지 변환 기능
    - 실시간 이미지 크기 조정과 포맷 변환
    - 자동 이미지 최적화와 압축 처리
    - 워터마크 적용과 이미지 필터링 옵션
- 백업과 복구 시스템
    - 자동화된 정기 백업과 버전 관리
    - 지역 간 데이터 복제와 재해 복구
    - 실수로 인한 삭제 방지와 복구 메커니즘

**구현 예시**

```javascript
const StorageService = {
  // 파일 업로드
  async uploadFile(bucket: string, path: string, file: File) {
    const { data, error } = await supabase
      .storage
      .from(bucket)
      .upload(path, file) // 오브젝트 스토리지로 업로드(자주 바뀌지 않는 데이터)
    return { data, error }
  },

  // 파일 다운로드
  async downloadFile(bucket: string, path: string) {
    const { data, error } = await supabase
      .storage
      .from(bucket)
      .download(path)
    return { data, error }
  },

  // 파일 목록 조회
  async listFiles(bucket: string, path: string) {
    const { data, error } = await supabase
      .storage
      .from(bucket)
      .list(path)
    return { data, error }
  }
}
```

### 3. PostgREST

**주요 특징**

- 자동 RESTful API 생성
    - 데이터베이스 스키마 기반의 API 엔드포인트 자동 구성
    - HTTP 메소드와 데이터베이스 작업의 직관적 매핑
    - 복잡한 라우팅 설정 없는 즉시 사용 가능한 API

```sql
-- 데이터베이스 테이블 생성 
CREATE TABLE products (
  id SERIAL PRIMARY KEY,  -- 자동 증가하는 기본키
  name TEXT,             -- 제품명을 저장하는 텍스트 필드
  price INTEGER          -- 가격을 저장하는 정수 필드
);

-- 자동으로 생성되는 API 엔드포인트:
GET /products         -- 모든 제품 목록을 가져옴
POST /products        -- 새로운 제품을 데이터베이스에 추가
PATCH /products?id=eq.1  -- ID가 1인 제품의 정보를 수정
DELETE /products?id=eq.1  -- ID가 1인 제품을 삭제
```

- 복잡한 쿼리 지원
    - 다중 테이블 조인과 서브쿼리의 URL 파라미터 변환
    - PostgreSQL의 고급 연산자와 함수 직접 활용
    - 수평/수직 필터링을 통한 데이터 요청 최적화

```sql
-- 다중 테이블 조인
GET /orders?select=id,total,customers(name)&total=gt.1000
-- 주문 테이블에서 총액이 1000 초과인 주문을 조회하고,
-- 각 주문과 연관된 고객의 이름도 함께 가져옴

-- 서브쿼리 활용
GET /products?price=lt.(select.avg.price.from.products)
-- 전체 제품의 평균 가격보다 낮은 가격의 제품들만 조회
```

- 관계형 데이터 조회
    - 단일 요청을 통한 중첩 관계 데이터 조회
    - 다대다 관계의 자동 처리와 데이터 구조화
    - 재귀적 관계 쿼리의 깊이 제어 옵션

```sql
-- 주문과 관련 고객 정보를 한 번에 조회
GET /orders?select=id,order_date,customer:customers(*)
-- 주문 정보와 함께 해당 주문을 한 고객의 모든 정보를 한 번에 가져옴

-- 중첩 관계 조회
GET /categories?select=name,products(name,price,reviews(rating,comment))
-- 카테고리별로 속한 제품들의 정보와
-- 각 제품에 달린 리뷰 정보까지 계층적으로 조회
```

- 필터링 및 정렬 기능
    - 다양한 비교 연산자를 활용한 동적 필터링
    - 복수 필드 기준의 정렬과 우선순위 지정
    - 전체 텍스트 검색과 패턴 매칭 지원

```sql
-- 다양한 필터링
GET /products?price=gt.1000&name=ilike.*phone*
-- 가격이 1000 초과이고 이름에 'phone'이 포함된 제품들을 조회

-- 복수 필드 정렬
GET /products?order=price.desc,name.asc
-- 제품을 가격 내림차순으로 정렬하고,
-- 같은 가격일 경우 이름 오름차순으로 정렬

-- 전체 텍스트 검색
GET /products?description=fts.smartphone
-- 설명에 'smartphone' 관련 단어가 포함된 제품을 전문 검색
```

- 페이지네이션 지원
    - 오프셋과 리밋 기반의 기본 페이지네이션
    - 커서 기반의 효율적인 대용량 데이터 페이징
    - 응답 헤더를 통한 페이지 메타데이터 제공

```sql
-- 기본 페이지네이션
GET /products?limit=10&offset=20
-- 21번째부터 30번째 제품까지 10개의 결과를 반환

-- 커서 기반 페이지네이션
GET /products?order=created_at.desc&limit=10
-- 생성일 기준으로 최신 제품 10개를 조회
// Response Header: Content-Range: 0-9/100
-- 전체 100개 중 0-9번째 결과를 반환했다는 의미
```

- 보안과 성능 최적화
    - RLS 정책과 완벽한 통합을 통한 데이터 보안
    - 쿼리 실행 계획의 자동 최적화
    - 요청 수 제한과 캐싱 메커니즘 내장

```sql
-- RLS 정책 설정
CREATE POLICY "Users can only see their own data"
ON orders FOR SELECT
USING (auth.uid() = user_id);
-- 사용자가 자신의 주문 데이터만 볼 수 있도록 제한하는 정책 설정

-- 자동 쿼리 최적화
GET /products?select=name,category(name)
-- 제품명과 해당 제품의 카테고리명을 조회
// PostgREST가 자동으로 효율적인 JOIN 쿼리 생성
-- PostgREST가 내부적으로 최적화된 JOIN 쿼리를 생성하여 성능 향상
```

**구현 예시**

```javascript
const APIService = {
  // 복잡한 관계형 쿼리
  async getComplexData() {
    const { data, error } = await supabase
      .from('posts')
      .select(`
        id,
        title,
        content,
        author:user_id(name, email),
        comments(
          id,
          content,
          user:user_id(name)
        )
      `)
      .eq('status', 'published')
      .order('created_at', { ascending: false })
      .range(0, 9)

    return { data, error }
  }
}
/* 변환되는 url로 http 요청 -> PostgREST가 이를 다시 SQL 쿼리로 변환
    GET https://[PROJECT_ID].supabase.co/rest/v1/posts
    ?select=id,title,content,author:user_id(name,email),comments(id,content,user:user_id(name))
    &status=eq.published
    &order=created_at.desc
    &offset=0
    &limit=9 
*/
```

### 4. Realtime Service

**주요 특징**

- WebSocket 기반 실시간 통신
    - 양방향 실시간 데이터 스트림의 효율적 처리
    - 자동 재연결과 상태 복구 메커니즘 내장
    - 네트워크 대역폭 최적화와 메시지 압축
- 데이터베이스 변경사항 자동 감지
    - PostgreSQL의 변경 사항 실시간 캡처와 전파
    - INSERT, UPDATE, DELETE 이벤트의 개별 구독
    - 변경 이력 추적과 순서 보장 메커니즘
- 채널 기반 구독 시스템
    - 테이블 또는 레코드 단위의 세분화된 구독
    - 사용자 정의 필터를 통한 이벤트 선별
    - 동적 구독 관리와 접근 제어
- 브로드캐스트 메시징
    - 채널별 실시간 메시지 브로드캐스팅
    - 사용자 그룹별 메시지 전송 기능
    - 대규모 동시 접속 처리와 부하 분산
- 자동 재연결 메커니즘
    - 네트워크 불안정 상황의 자동 감지와 복구
    - 지수 백오프 알고리즘을 통한 재연결 시도
    - 연결 상태 모니터링과 디버깅 도구
- 보안과 인증
    - 채널별 접근 권한의 세밀한 제어
    - JWT 기반 인증과 실시간 토큰 검증
    - 클라이언트 연결 수 제한과 리소스 관리

**구현 예시**

```javascript
const RealtimeService = {
  // 테이블 변경사항 구독
  subscribeToChanges(table: string, callback: Function) {
    const subscription = supabase
      .channel(`public:${table}`)
      .on('postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: table
        },
        (payload) => {
          callback(payload)
        }
      )
      .subscribe()

    return subscription
  },

  // 특정 레코드 구독
  subscribeToRecord(table: string, id: number, callback: Function) {
    const subscription = supabase
      .channel(`public:${table}:${id}`)
      .on('postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: table,
          filter: `id=eq.${id}`
        },
        (payload) => {
          callback(payload)
        }
      )
      .subscribe()

    return subscription
  }
}
```

### 5. Edge Functions

**주요 특징**

- Deno 런타임 환경
    - 내장된 TypeScript 지원과 즉시 실행
    - 보안 샌드박스 모델을 통한 격리 실행
    - 웹 표준 API의 네이티브 지원
- TypeScript/JavaScript 지원
    - 타입 안정성이 보장된 서버리스 함수 작성
    - 최신 ECMAScript 기능의 즉시 활용
    - 광범위한 NPM 패키지 생태계 접근
- 글로벌 배포 인프라
    - 전세계 엣지 로케이션을 통한 즉시 배포
    - 사용자 위치 기반의 자동 라우팅
    - 글로벌 분산 실행을 통한 지연 시간 최소화
- 자동 스케일링
    - 트래픽 부하에 따른 자동 확장과 축소
    - 서버리스 아키텍처의 자원 최적화
    - 무중단 배포와 버전 관리 시스템
- 환경 변수 관리
    - 암호화된 환경 변수의 안전한 저장
    - 개발/스테이징/프로덕션 환경 설정 분리
    - 프로젝트별 시크릿 키 관리 시스템
- 외부 API 통합
    - 내장 HTTP 클라이언트를 통한 외부 서비스 연동
    - 요청/응답의 자동 직렬화와 역직렬화
    - API 응답 캐싱과 성능 최적화
- 모니터링과 로깅
    - 실시간 함수 실행 상태 모니터링
    - 상세한 에러 추적과 디버깅 정보
    - 사용량 통계와 성능 메트릭 수집

**구현 예시**

```javascript
// /functions/process-payment.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'

// 결제 처리 Edge Function
serve(async (req) => {
  const { amount, currency } = await req.json()

  // 결제 처리 로직
  const paymentResult = await processPayment(amount, currency)

  return new Response(
    JSON.stringify(paymentResult),
    { headers: { 'Content-Type': 'application/json' } }
  )
})

// 클라이언트 호출 유틸리티
const EdgeFunctionService = {
  async callEdgeFunction(functionName: string, body: any) {
    const { data, error } = await supabase.functions.invoke(functionName, {
      body: JSON.stringify(body)
    })
    return { data, error }
  }
}
```

**Edge Functions 활용 사례**

- 결제 처리
    - 다양한 결제 게이트웨이와의 안전한 통합
    - 결제 검증과 환불 프로세스의 자동화
    - 다중 통화 지원과 환율 자동 적용
- 이메일 발송
    - 트리거 기반의 자동화된 이메일 발송
    - 커스텀 템플릿과 동적 콘텐츠 생성
    - 대량 메일링과 발송 상태 추적
- 이미지 처리
    - 실시간 이미지 리사이징과 포맷 변환
    - 워터마크 적용과 이미지 최적화
    - AI 기반 이미지 분석과 메타데이터 추출
- 외부 API 통합
    - 서드파티 서비스와의 실시간 데이터 동기화
    - API 응답의 캐싱과 속도 최적화
    - 다중 API 호출의 병렬 처리
- 복잡한 비즈니스 로직
    - 맞춤형 데이터 검증과 변환 처리
    - 워크플로우 자동화와 상태 관리
    - 복잡한 계산과 데이터 집계 처리
- 배치 작업 처리
    - 정기적인 데이터 정리와 최적화
    - 대량 데이터 처리와 ETL 작업
    - 예약된 작업 실행과 결과 보고

## Part 3: Connection Pooler & Row Level Security

### 1. Connection Pooler (PgBouncer)

**주요 특징**

- 데이터베이스 연결 최적화
    - 제한된 데이터베이스 연결의 효율적 관리
    - 동시 접속자 수 증가에 따른 확장성 제공
    - 연결 생성/해제 오버헤드 최소화
- 리소스 사용 효율화
    - 메모리와 CPU 사용량의 최적화
    - 유휴 연결의 효율적 재사용
    - 시스템 리소스 부하 분산
- 자동 연결 관리
    - 연결 풀의 자동 확장과 축소
    - 비정상 연결의 자동 감지와 복구
    - 연결 타임아웃 설정과 관리

**풀링 종류**

- Transaction 모드
    - 트랜잭션 단위의 연결 할당과 반환
    - 최소한의 리소스로 최대 성능 확보
    - CRUD 작업에 최적화된 연결 관리
- Statement 모드
    - 개별 SQL 문장별 연결 재사용
    - 단순 쿼리 처리의 성능 최적화
    - 최대 연결 공유율 달성
- Session 모드
    - 세션 기반의 지속적 연결 유지
    - 세션 변수와 임시 테이블 지원
    - Prepared Statement의 효율적 활용

※ Prepared Statement: SQL 쿼리의 템플릿을 미리 준비해두고 실행 시점에 파라미터만 바인딩하는 데이터베이스 기능

Transaction이나 Statement 모드에서는 연결이 지속적으로 재사용되므로 Prepared Statement를 유지할 수 없음

```sql
-- 쿼리 템플릿 준비
PREPARE user_query(text) AS
SELECT * FROM users WHERE name = $1;

-- 실제 실행 시 파라미터만 전달
EXECUTE user_query('John');
```

**구현 예시**

```javascript
// Connection Pool 설정
const poolConfig = {
  mode: 'transaction',
  max: 10,
  min: 2,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
}

const supabase = createClient('URL', 'ANON_KEY', {
  db: { poolConfig }
})
```

### 2. Row Level Security (RLS)

**주요 특징**

- 데이터베이스 수준 보안
    - 테이블 단위의 세밀한 접근 제어 구현
    - 데이터베이스 레벨에서의 보안 정책 적용
    - 애플리케이션 로직과 독립된 보안 계층
- 행 단위 접근 제어
    - 개별 레코드 수준의 권한 관리
    - 사용자별 데이터 접근 범위 제한
    - 동적 필터링을 통한 데이터 노출 제어
- 정책 기반 보안 규칙
    - SQL 표현식을 활용한 유연한 정책 정의
    - 다양한 비즈니스 규칙의 보안 정책화
    - 정책의 조합과 우선순위 관리
- JWT 토큰 통합
    - 인증 토큰 기반의 사용자 식별
    - 토큰 클레임을 활용한 권한 검증
    - 실시간 권한 변경 적용

**구현 예시**

```sql
-- 기본 테이블 설정
CREATE TABLE private_documents (
  id SERIAL PRIMARY KEY,
  title TEXT,
  content TEXT,
  user_id UUID REFERENCES auth.users,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS 활성화
ALTER TABLE private_documents ENABLE ROW LEVEL SECURITY;

-- 기본 정책들
CREATE POLICY "사용자 문서 조회" ON private_documents
FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "사용자 문서 생성" ON private_documents
FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "사용자 문서 수정" ON private_documents
FOR UPDATE USING (auth.uid() = user_id);

-- 복잡한 정책 예시
CREATE POLICY "팀 문서 공유" ON team_documents
FOR SELECT USING (
  auth.uid() IN (
    SELECT user_id
    FROM team_members
    WHERE team_id = team_documents.team_id
  )
);
```

**RLS 정책 유형**

- 읽기 정책 (SELECT)
    - 조회 권한의 세밀한 제어
    - 사용자별 데이터 필터링
    - 민감 정보 접근 제한
- 쓰기 정책 (INSERT/UPDATE/DELETE)
    - 데이터 수정 권한의 제어
    - 사용자별 데이터 생성 제한
    - 삭제 작업의 안전성 보장
- 보안 컨텍스트 기능
    - 현재 사용자 컨텍스트 기반 제어
    - 조직/그룹별 데이터 접근 관리
    - 시간 기반 접근 제한 구현
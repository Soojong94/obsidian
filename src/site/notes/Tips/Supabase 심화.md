---
{"dg-publish":true,"permalink":"/tips/supabase/"}
---

# 심화 Supabase 기능 가이드: 통합 예시 및 실제 활용

## 1. 통합 애플리케이션 예시

### 문서 관리 시스템 구현 예시

```typescript
// 문서 기본 구조 정의
interface Document {
 id: number;            // 문서 고유 ID
 title: string;         // 문서 제목
 content: string;       // 문서 내용
 version: number;       // 문서 버전 관리용 번호
 status: 'draft' | 'published' | 'archived';  // 문서 상태
 created_by: string;    // 작성자 ID
 created_at: string;    // 생성 시간
 updated_at: string;    // 최종 수정 시간
 tags: string[];        // 문서 태그 목록
}

class DocumentManager {
 private supabase: SupabaseClient;

 constructor() {
   // Supabase 클라이언트 초기화
   this.supabase = createClient(
     process.env.SUPABASE_URL!,
     process.env.SUPABASE_ANON_KEY!
   );
 }

 // 새 문서 생성 메서드
 async createDocument(title: string, content: string, tags: string[] = []): Promise<Document> {
   const { data, error } = await this.supabase
     .from('documents')
     .insert({
       title,
       content,
       version: 1,        // 최초 버전은 1로 설정
       status: 'draft',   // 초기 상태는 draft로 설정
       tags
     })
     .select()
     .single();

   if (error) throw error;
   return data;
 }

 // 문서 내용 업데이트 메서드
 async updateDocument(documentId: number, content: string): Promise<Document> {
   const { data, error } = await this.supabase
     .from('documents')
     .update({
       content,
       version: sql`version + 1`,  // 버전 자동 증가
       updated_at: new Date().toISOString()
     })
     .eq('id', documentId)
     .select()
     .single();

   if (error) throw error;
   return data;
 }

 // 실시간 문서 변경 구독 메서드
 subscribeToDocument(documentId: number, callback: (payload: any) => void) {
   return this.supabase
     .channel(`document:${documentId}`)
     .on('postgres_changes', 
       {
         event: '*',             // 모든 이벤트 감지
         schema: 'public',
         table: 'documents',
         filter: `id=eq.${documentId}`  // 특정 문서만 필터링
       },
       callback
     )
     .subscribe();
 }

 // 문서 첨부파일 업로드 메서드
 async uploadAttachment(documentId: number, file: File) {
   // 파일 경로 생성 (timestamp로 고유성 보장)
   const filePath = `documents/${documentId}/${Date.now()}_${file.name}`;
   
   const { error: uploadError } = await this.supabase
     .storage
     .from('document-attachments')    // document-attachments 버킷 사용
     .upload(filePath, file, {
       cacheControl: '3600',          // 1시간 캐시 설정
       upsert: true                   // 같은 경로 파일 덮어쓰기 허용
     });

   if (uploadError) throw uploadError;
   return filePath;
 }
}
```

## 2. 최적화 전략

### 데이터베이스 최적화

- 인덱스 활용
- 적절한 RLS 정책 설계
- 효율적인 쿼리 작성
- Connection Pool 설정 최적화

### 실시간 기능 최적화

- 필요한 데이터만 구독
- 적절한 필터링 적용
- 효율적인 이벤트 처리
- 재연결 전략 구현

### 스토리지 최적화

- CDN 활용
- 이미지 변환 활용
- 적절한 청크 사이즈 설정
- 파일 메타데이터 관리

### 구현 예시

```typescript
// 데이터베이스 최적화 설정 예시
/*
-- 전문 검색을 위한 인덱스 생성
CREATE INDEX idx_documents_search 
ON documents USING GIN (to_tsvector('english', content));

-- 문서 접근 제어를 위한 RLS 정책
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- 조회 권한 정책 설정
CREATE POLICY "공개 문서 조회" ON documents
FOR SELECT USING (
 status = 'published' OR   -- 공개된 문서이거나
 auth.uid() = created_by   -- 작성자 본인인 경우
);
*/

// 실시간 기능 사용 예시
const subscribeToPublicDocuments = () => {
 supabase
   .channel('documents')
   .on('postgres_changes', 
     {
       event: 'UPDATE',              // 업데이트 이벤트만 구독
       schema: 'public',
       table: 'documents',
       filter: 'status=eq.published' // 공개 문서만 필터링
     },
     (payload) => {
       console.log('문서 업데이트:', payload);
     }
   )
   .subscribe();
};

// 스토리지 최적화 예시
const getOptimizedImageUrl = (path: string) => {
 // Supabase Storage 변환 기능을 사용한 이미지 최적화
 return supabase
   .storage
   .from('images')
   .getPublicUrl(path, {
     transform: {
       width: 800,         // 너비 조정
       height: 600,        // 높이 조정
       quality: 80         // 품질 설정
     }
   });
};
```

## 3. 보안 고려사항

### 인증 보안

- 적절한 토큰 만료 시간
- 안전한 비밀번호 정책
- 2FA 구현
- 세션 관리

### 데이터 보안

- RLS 정책 철저한 검토
- API 접근 제한
- 암호화 적용
- 백업 전략

### Edge Function 보안

- 환경 변수 관리
- 접근 제어
- 입력값 검증
- 에러 처리

### 구현 예시

```typescript
// 인증 설정
const authConfig = {
  auth: {
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: true
  }
};

const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY, authConfig);

// 2FA 활성화 - Supabase 대시보드에서 설정
// RLS 정책 - SQL로 정책만 정의
// 백업 - Supabase 대시보드에서 스케줄링

// Edge Function에서 필요한 최소한의 보안 처리
export const myFunction = async (req: any) => {
  try {
    // 입력값 검증
    const { data } = req.body;
    if (!data) throw new Error('Invalid input');

    // 비즈니스 로직
    const result = await processData(data);
    
    return { data: result };
  } catch (error) {
    return { error: error.message };
  }
};
```
# API 명세서: RFPilot POC

> 📌 실시간 인터랙티브 명세는 FastAPI Swagger UI 참조
> → http://localhost:8000/docs
> 이 문서는 설계 단계 요약본 + 미팅 공유용.

## 1. 기본 정보

| 항목 | 값 |
|---|---|
| Base URL (개발) | `http://localhost:8000` |
| Base URL (운영) | (본 프로젝트에서 결정) |
| 인증 방식 | (POC — 인증 없음) |
| 응답 포맷 | JSON |
| 문자 인코딩 | UTF-8 |
| CORS 허용 | `http://localhost:3000` (React) |

## 2. 공통 응답 구조

### 성공 응답
​```json
{
  "status": "success",
  "data": { ... },
  "message": "정상 처리"
}
​```

### 실패 응답
​```json
{
  "status": "error",
  "error_code": "INVALID_INPUT",
  "message": "입력값이 올바르지 않습니다"
}
​```

## 3. 엔드포인트 목록

| Method | 경로 | 설명 | 우선순위 |
|---|---|---|---|
| GET | `/` | 서버 헬스체크 | High |
| POST | `/upload` | PDF 업로드 + 임베딩 | High |
| POST | `/chat` | 챗봇 질의응답 | High |
| POST | `/reload-model` | 모델 재로딩 (관리용) | Low |

---

### 3.1 GET `/` — 헬스체크

**설명**: 서버가 정상 작동 중인지 확인 (React에서 페이지 진입 시 호출)

**요청**: 파라미터 없음

**응답 예시**
​```json
{
  "status": "success",
  "message": "RFPilot FastAPI 서버 정상 작동 중!",
  "version": "0.1.0"
}
​```

---

### 3.2 POST `/upload` — PDF 업로드

**설명**: PDF 파일 업로드 → 텍스트 추출 → 청크 분할 → 임베딩 → FAISS 저장

**요청 (multipart/form-data)**

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `file` | File | ✅ | PDF 파일 (최대 10MB) |
| `session_id` | string | ✅ | 사용자 세션 식별자 (UUID 권장) |

**응답 예시 (성공)**
​```json
{
  "status": "success",
  "data": {
    "session_id": "user-uuid-12345",
    "filename": "RFP_샘플.pdf",
    "page_count": 12,
    "chunk_count": 32,
    "processing_time_ms": 6420
  },
  "message": "분석 완료"
}
​```

**응답 예시 (실패)**
​```json
{
  "status": "error",
  "error_code": "FILE_TOO_LARGE",
  "message": "파일 크기가 10MB를 초과합니다"
}
​```

**에러 코드**

| 코드 | HTTP | 상황 |
|---|---|---|
| `FILE_TOO_LARGE` | 413 | 10MB 초과 |
| `INVALID_FILE_TYPE` | 400 | PDF가 아닌 파일 |
| `FILE_CORRUPTED` | 400 | PDF 파싱 실패 |
| `EMPTY_FILE` | 400 | 빈 파일 |
| `INTERNAL_ERROR` | 500 | 서버 오류 |

---

### 3.3 POST `/chat` — 챗봇 질의응답

**설명**: 사용자 질문 → 관련 청크 검색 → LLM 답변 생성

**요청 (application/json)**
​```json
{
  "session_id": "user-uuid-12345",
  "message": "이 제안서의 사업 기간은?"
}
​```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `session_id` | string | ✅ | `/upload`에서 받은 세션 ID |
| `message` | string | ✅ | 사용자 질문 (최대 1000자) |

**응답 예시 (성공)**
​```json
{
  "status": "success",
  "data": {
    "answer": "본 사업의 계약 기간은 계약 후 6개월입니다. 제안서 3페이지에 명시되어 있습니다.",
    "session_id": "user-uuid-12345",
    "retrieved_chunks": 3,
    "response_time_ms": 3120
  }
}
​```

**응답 예시 (실패)**
​```json
{
  "status": "error",
  "error_code": "SESSION_NOT_FOUND",
  "message": "먼저 PDF를 업로드해주세요"
}
​```

**에러 코드**

| 코드 | HTTP | 상황 |
|---|---|---|
| `EMPTY_MESSAGE` | 400 | message 빈 문자열 |
| `MESSAGE_TOO_LONG` | 400 | message 1000자 초과 |
| `SESSION_NOT_FOUND` | 404 | session_id 없음 (PDF 미업로드) |
| `MODEL_ERROR` | 500 | LLM 호출 실패 |
| `EMBEDDING_ERROR` | 500 | 임베딩 호출 실패 |
| `INTERNAL_ERROR` | 500 | 그 외 서버 오류 |

---

### 3.4 POST `/reload-model` — 모델 재로딩 (관리용)

**설명**: 운영 중 모델 재로딩 (POC에선 더미 응답)

**요청**: 파라미터 없음

**응답 예시 (POC 단계)**
​```json
{
  "status": "success",
  "message": "[POC] 모델 리로드 구현 예정"
}
​```

> ⚠️ POC 단계에선 더미 응답만 반환. 본 프로젝트에서 실제 구현.

---

## 4. 데이터 모델 (Pydantic)

본 절은 노수민님이 코드 짤 때 그대로 참고 가능.

​```python
# src/schemas/chat.py

from pydantic import BaseModel, Field

class ChatRequest(BaseModel):
    session_id: str = Field(..., description="사용자 세션 ID")
    message: str = Field(..., max_length=1000, description="사용자 질문")

class ChatResponse(BaseModel):
    answer: str
    session_id: str
    retrieved_chunks: int
    response_time_ms: int

class UploadResponse(BaseModel):
    session_id: str
    filename: str
    page_count: int
    chunk_count: int
    processing_time_ms: int

class ErrorResponse(BaseModel):
    status: str = "error"
    error_code: str
    message: str
​```

## 5. 주의사항

- **Swagger UI가 진실의 원천**: 본 문서와 Swagger UI 내용이 다르면 Swagger UI를 우선
- **변경 시 본 문서도 같이 업데이트**: API 변경할 때 이 문서 같이 손봐주세요
- **본 프로젝트 이월 항목**: 인증, rate limit, API 버전 관리, 상세 로깅

## 6. 작성 이력

| 날짜 | 작성자 | 변경 내용 |
|---|---|---|
| 2026-06-04 | 노수민 | 초안 작성, `/`, `/upload`, `/chat` 명세 |
| | | |

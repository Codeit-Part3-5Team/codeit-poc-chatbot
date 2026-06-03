# API 명세서

> 📌 실시간 인터랙티브 명세는 FastAPI Swagger UI 참조  
> → http://localhost:8000/docs  
> 이 문서는 설계 단계 요약본입니다.

## 1. 기본 정보

| 항목 | 값 |
|---|---|
| Base URL | `http://localhost:8000` |
| 인증 방식 | (POC 단계 — 인증 없음) |
| 응답 포맷 | JSON |
| 문자 인코딩 | UTF-8 |

## 2. 공통 응답 구조

**성공 응답**
```json
{
  "status": "success",
  "data": { ... },
  "message": "정상 처리"
}
```

**실패 응답**
```json
{
  "status": "error",
  "error_code": "INVALID_INPUT",
  "message": "입력값이 올바르지 않습니다"
}
```

## 3. 엔드포인트 목록

| Method | 경로 | 설명 | 담당 |
|---|---|---|---|
| GET | `/` | 서버 헬스체크 | 노수민 |
| POST | `/chat` | 챗봇 질의응답 | 노수민 |
| POST | `/reload-model` | 모델 재로딩 (관리용) | 노수민 |

---

### 3.1 GET `/` — 헬스체크

**설명**: 서버가 정상 작동 중인지 확인

**요청**: 파라미터 없음

**응답 예시**
```json
{
  "status": "success",
  "message": "Codeit AI FastAPI 서버 정상 작동 중!",
  "version": "1.0"
}
```

---

### 3.2 POST `/chat` — 챗봇 질의응답

**설명**: 사용자 질문에 대한 챗봇 답변 반환

**요청 (Request Body)**
```json
{
  "session_id": "user-uuid-12345",
  "message": "환불 정책 알려줘"
}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| session_id | string | ✅ | 사용자 세션 식별자 |
| message | string | ✅ | 사용자 질문 (최대 1000자) |

**응답 예시 (성공)**
```json
{
  "status": "success",
  "data": {
    "answer": "환불은 구매 후 7일 이내 가능합니다.",
    "session_id": "user-uuid-12345",
    "response_time_ms": 2340
  }
}
```

**응답 예시 (실패)**
```json
{
  "status": "error",
  "error_code": "EMPTY_MESSAGE",
  "message": "질문 내용이 비어 있습니다"
}
```

**에러 코드 목록**

| 코드 | HTTP | 상황 |
|---|---|---|
| `EMPTY_MESSAGE` | 400 | message가 빈 문자열 |
| `MESSAGE_TOO_LONG` | 400 | message가 1000자 초과 |
| `SESSION_NOT_FOUND` | 404 | session_id 미존재 |
| `MODEL_ERROR` | 500 | LLM 호출 실패 |
| `INTERNAL_ERROR` | 500 | 그 외 서버 오류 |

---

### 3.3 POST `/reload-model` — 모델 재로딩

**설명**: 운영 중 모델을 재로딩 (관리자용, POC에선 테스트 목적)

**요청**: 파라미터 없음

**응답 예시**
```json
{
  "status": "success",
  "message": "모델 리로드 완료"
}
```

> ⚠️ POC 단계에선 더미 응답만 반환, 실제 리로드는 추후 구현 예정

---

## 4. 작성 이력

| 날짜 | 작성자 | 변경 내용 |
|---|---|---|
| 2026-06-04 | 노수민 | 초안 작성 |
| | | |
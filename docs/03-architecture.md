# 시스템 아키텍처

## 1. 구성도
[사용자 브라우저]
       ↓ HTTPS
[React Frontend (localhost:3000)]
       ↓ REST API (JSON)
[FastAPI Backend (localhost:8000)]
       ├→ [LLM API: OpenAI / HuggingFace]
       ├→ [벡터 DB: FAISS / Chroma]
       └→ [세션 저장소: 인메모리 / Redis]

## 2. 주요 흐름
1. 사용자가 React UI에 질문 입력
2. React → POST /chat → FastAPI
3. FastAPI: 벡터DB 검색 → LLM 호출 → 답변 생성
4. FastAPI → JSON 응답 → React 렌더링

## 3. 기술 선택 이유
- FastAPI: 비동기 처리, 자동 API 문서화
- React: (선택 이유)
- (그 외)
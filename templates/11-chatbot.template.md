# 🌐 서비스 개발 및 EL 담당 전용 소프트웨어 개발 계획서 (SDP)
## B2G RFP RAG 시스템 — Service Dev & EL Team

> **프로젝트명**: 입찰메이트 사내 RAG 시스템 구축 (B2G 입찰지원 컨설팅)  
> **문서 버전**: v1.0  
> **작성일**: 2026-06-18  
> **작성자**: 전민재 PM  
> **대상 팀**: 서비스 개발 및 EL 담당 (노수민님)  
> **프로젝트 기간**: 2026-06-19(금) ~ 2026-06-26(금) — 총 8일

---

## 1. 프로젝트 개요 (Project Overview)

### 1.1 프로젝트 배경
- '입찰메이트' 사내 RAG 시스템의 **사용자 접점(End-User Layer)** 구축.
- 컨설턴트가 자연어 질의로 RFP 정보를 받아볼 수 있는 **웹 챗봇 프로그램** 제공.
- R/G 팀이 만든 RAG 파이프라인을 **외부로 노출**하는 게이트웨이 역할.

### 1.2 서비스 개발 & EL의 미션
> **"R/G 팀이 만든 RAG 엔진을 컨설턴트가 사용할 수 있는 웹 챗봇으로 만든다."**  
> **"백엔드 1줄(`get_ai_response()`) 교체만으로 LLM/RAG 스왑이 가능한 모듈화 구조를 유지한다."**

### 1.3 핵심 설계 원칙 (Modularization)
> **현재**: Groq API 직접 호출 (하드코딩) ← 프로토타입  
> **목표**: 전민재 PM 프로토타입 기반 **모듈화 구조로 전환**
> - LLM 교체 가능 (Groq → 팀 파인튜닝 모델 → Ollama)
> - 프롬프트 템플릿 시스템 (도메인 교체 가능)
> - RAG 워크플로우 연동 (벡터DB 완성 후)
> - 대화 이력 요약 (단순 잘라내기 → LLM 요약)

### 1.4 최종 목표 (Definition of Done)
| # | 산출물 | 완료 기준 |
|---|--------|----------|
| 1 | 웹 챗봇 프론트엔드 | 메시지 입출력, 채팅 UI 정상 동작 |
| 2 | 백엔드 API (`get_ai_response()`) | R/G 팀 파이프라인 호출, JSON 응답 반환 |
| 3 | 모듈화된 Generator 구조 | LLM 클라이언트/프롬프트/메모리 분리 |
| 4 | RAG 파이프라인 연동 | Naive + Agentic 모두 호출 가능 |
| 5 | 대화 히스토리 관리 | 최근 N턴 유지 (슬라이더 UI 포함) |
| 6 | UX 개선 (최소) | 자동 스크롤, 에러 메시지, 입력창 비활성화 |
| 7 | (스트레치) GCP Ollama 연동 | gpt-5-mini와 토글로 전환 |

---

## 2. 범위 정의 (Scope)

### 2.1 ✅ 포함 범위 (In-Scope)

**프론트엔드 (웹 챗봇)**
- 채팅 UI (메시지 버블, 사용자/AI 구분)
- 메시지 입력창 + 전송 버튼
- 자동 스크롤 (새 메시지 하단으로)
- 입력창 비활성화 (응답 대기 중)
- 에러 메시지 고도화 (네트워크/API 실패 표시)
- 메시지 복사 버튼
- 대화 이력 제한 슬라이더 (구현 완료 — 활용 강화)
- 응답 출처(sources) 표시 영역

**백엔드 (API + 통합)**
- 메인 함수: `get_ai_response(query, history)`
- R/G 팀의 `pipeline.py` import 및 호출
- 단순 대화 이력 관리 (메모리 내, 단일 세션)
- 에러 핸들링 (API 실패, 타임아웃)
- 응답 시간 측정/로깅

**LLM 라우팅 (EL: Endpoint Layer)**
- 1순위: OpenAI gpt-5-mini (R/G 팀 파이프라인 경유)
- 2순위 (스트레치): GCP Ollama 직접 연동 + 토글

### 2.2 ❌ 미포함 범위 (Out-of-Scope) — 가이드에 명시된 백로그
- **로그인 / JWT 인증**
- **사용자별 대화 저장** (PostgreSQL 등 별도 DB)
- 다크모드, 마크다운 렌더링, 모바일 반응형 — 시간 남으면 추가
- 음성 입출력 (STT/TTS)
- 다국어 지원
- 도커 컨테이너화 / Fast API 풀스택 서빙 (가이드: Part 4 이후)

> **⚠️ Scope Creep 방지**: "다크모드 예쁘게 추가하자"는 의견은 백로그.  
> 1인이 8일 동안 풀스택 + 통합까지 해야 함을 명심.

### 2.3 ⚖️ 프레임워크 선택 (중요한 의사결정)

> **권장: Streamlit** (1인 개발 + 8일 + 초급 → 가장 현실적)

| 프레임워크 | 장점 | 단점 | 추천도 |
|-----------|------|------|--------|
| **Streamlit** ⭐ | 1일에 챗봇 골격, 풀스택 1인 가능, Python만 | 디자인 한계 | **1순위** |
| Gradio | ChatInterface 컴포넌트 좋음 | 커스터마이징 약함 | 2순위 |
| React + FastAPI | 디자인 자유도 高 | 학습 비용 高, 1인 8일 무리 | 비추천 |

> **확정 권장**: Streamlit으로 시작 → 시간 남으면 UX 다듬기.  
> 만약 기존 프로토타입이 React 등이면 그대로 유지하되, 백엔드 모듈화에 집중.

---

## 3. 방법론 (Methodology)

### 3.1 1인 개발 특수성
- 1인 개발 + 8일 → **모놀리식 코드라도 동작 먼저, 리팩토링 나중**
- 매일 **데일리 스크럼 참여 필수** (R/G/데이터 팀 소통 채널)
- 코드 리뷰는 전민재 PM 또는 R/G 팀원에게 비동기로 요청 (Discord)

### 3.2 협업 규칙
| 항목 | 규칙 |
|------|------|
| 브랜치 | `feat/chatbot-*` |
| PR 리뷰 | 전민재 PM 또는 R/G 팀 1인 |
| 일일 머지 | `feat/chatbot-` 브랜치에 매일 push - 전민재 PM PR(Pull Request) 업무 진행 |
| 실험 노트 | Notion에 UI/UX 결정 기록 |
| 커밋 컨벤션 | `[Service] feat: 자동 스크롤`, `[EL] fix: 에러 핸들링` |

### 3.3 인터페이스 우선 원칙 (Interface-First)
> R/G 팀 결과물이 늦어도 **모킹(mocking)**으로 먼저 개발 가능하도록 설계.

```python
# Sprint 0~1: mock 인터페이스로 UI 먼저 완성
def get_ai_response_MOCK(query, history=None):
    return {
        "answer": f"[MOCK] {query}에 대한 답변입니다.",
        "sources": [{"doc_id": "DOC_001", "page": 1, "score": 0.9}],
        "retrieved_chunks": ["샘플 청크"],
        "elapsed_sec": 1.5,
        "tokens_used": 100
    }

# Sprint 2: 실제 R/G 팀 파이프라인으로 1줄 교체
from rag.pipeline import get_ai_response
```

---

## 4. 일정 및 마일스톤 (Schedule & Milestones)

### 4.1 전체 타임라인 (8일)

| 일자 | 요일 | 단계 | 주요 마일스톤 |
|------|------|------|--------------|
| 06/19 | 금 | **셋업** | 환경 구축 + 소프트웨어 개발 계획서 문서 분석 |
| 06/20 | 토 | **UI 골격** | 챗봇 UI v0.1 (mock 응답으로 동작) |
| 06/21 | 일 | **모듈화** | LLM/Prompt/Memory 분리, get_ai_response() 구조 확정 |
| 06/22 | 월 | **M1: UI 완성** | ⭐ UI v1.0 (mock 기반 풀 스택 동작) + KPT 회고 |
| 06/23 | 화 | **RAG 통합** | R/G 팀 파이프라인 import, mock → 실제 전환 |
| 06/24 | 수 | **UX 다듬기** | 자동스크롤, 에러처리, 복사버튼, 출처 표시 |
| 06/25 | 목 | **M2: 통합 완료** | ⭐ E2E 동작 + Ollama 토글(시간 시) + KPT 회고 |
| 06/26 | 금 | **Wrap-up** | 데모 영상, 발표자료 서비스 파트, 보고서 |

### 4.2 일별 상세 작업

#### 🔹 06/19 (금) - Sprint 0: 셋업
| 시간대 | 작업 | 산출물 |
|--------|------|--------|
| 오전 | Python 3.12.3 + 환경 셋업, OT 참여 | 환경 |
| 오후 | 소프트웨어 개발 계획서 내용 분석 | 업무 구체화 |
| 오후 | 기존 프로토타입 코드 리뷰 (Groq 하드코딩 부분 파악) | 분석 노트 |
| 저녁 | 백엔드 구조 다이어그램 설계 (Notion) | 아키텍처 |

#### 🔹 06/20 (토) - UI 골격
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| `app.py`: React 채팅 UI 기본 (`st.chat_message`, `st.chat_input`) | 0.3일 | `app.py` |
| Mock `get_ai_response()` 함수 구현 | 0.2일 | `service/mock.py` |
| 세션 상태 관리 (`st.session_state.messages`) | 0.3일 | UI 상태 |
| 응답 출처 표시 영역 (expander) | 0.2일 | UI |

#### 🔹 06/21 (일) - 모듈화 (가이드의 "Generator 구조 고도화" 반영)
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| `service/llm_client.py`: LLM 호출 추상화 (OpenAI/Ollama 분기) | 0.5일 | `llm_client.py` |
| `service/prompt_templates.py`: 도메인별 프롬프트 분리 | 0.3일 | `prompt_templates.py` |
| `service/memory.py`: 대화 이력 관리 (단순 잘라내기 v0.1) | 0.3일 | `memory.py` |
| `service/pipeline.py`: `get_ai_response()` 메인 함수 (mock 사용) | 0.4일 | `pipeline.py` |

#### 🔹 06/22 (월) - M1: UI v1.0 완성
| 작업 | 산출물 |
|------|--------|
| 오전: 모듈화된 코드 + UI 통합 검증 | 통합 동작 |
| 오후: R/G 팀과 `get_ai_response()` 인터페이스 최종 확정 | 인터페이스 문서 |
| 오후: 대화 이력 제한 슬라이더 UI 강화 | UI |
| ⭐ **M1 완료**: mock 기반 풀스택 E2E 동작 | M1 검증 |
| 저녁: KPT 회고 (전민재 PM과 1:1) | 회고록 |

#### 🔹 06/23 (화) - RAG 통합
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| R/G 팀 `pipeline.py` import 및 mock 교체 | 0.3일 | 통합 코드 |
| 응답 형식 매핑 (sources, retrieved_chunks 등) | 0.3일 | 매핑 코드 |
| 응답 시간 측정 / 표시 | 0.2일 | 타이머 |
| 통합 테스트 5건 + 버그 픽스 | 0.5일 | 테스트 로그 |

#### 🔹 06/24 (수) - UX 다듬기
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| 자동 스크롤 (`st.chat_message` 자동 처리됨, 확인) | 0.2일 | UI |
| 입력창 비활성화 (응답 대기 중) | 0.2일 | UI |
| 에러 메시지 고도화 (try-except + 사용자 친화 메시지) | 0.3일 | 에러 핸들링 |
| 메시지 복사 버튼 (`st.code` 또는 커스텀) | 0.2일 | UI |
| 출처 클릭 시 청크 미리보기 | 0.3일 | UI |

#### 🔹 06/25 (목) - M2: 통합 완료
| 작업 | 산출물 |
|------|--------|
| 오전: Ollama 연동 시도 (GCP VM의 Ollama 서버 + LangChain) | Ollama 클라이언트 |
| 오전: LLM 토글 UI (OpenAI ↔ Ollama) — 시간 부족 시 백로그 | (조건부) |
| 오후: 전체 시나리오 테스트 (단일/다중/후속/거부) | 테스트 결과 |
| ⭐ **M2 완료**: 모든 핵심 기능 E2E 동작 | M2 검증 |
| 저녁: KPT 회고 | 회고록 |

#### 🔹 06/26 (금) - Wrap-up
| 작업 | 산출물 |
|------|--------|
| 데모 영상 녹화 (3분 내외, 시나리오 4종 시연) | 데모 영상 |
| 서비스 README 작성 (실행 방법, 아키텍처 다이어그램) | `README.md` |
| 발표 자료 서비스 파트 작성 (3~4슬라이드) | 슬라이드 |
| 보고서 서비스/EL 파트 통합 | 보고서 일부 |

### 4.3 1인 개발자 일일 작업량 가이드
- 하루 순수 코딩 시간 **5~6시간** 가정 (혼자라 회의 적음)
- 막힐 때 30분 이상 끌지 말 것 → 즉시 Discord에 도움 요청
- 매일 17:00 `feat/chatbot-*` 브랜치 push (혼자라도 백업 차원)

---

## 5. 자원 및 역할 (Resources & Roles)

### 5.1 인력
| 역할 | 담당자 | R&R |
|------|--------|-----|
| **Service Dev & EL Engineer** | TBD (1인 개발) | 프론트엔드 UI, 백엔드 API, RAG 통합, Ollama 연동 |
| **전민재 PM (지원)** | 팀장 | R/G 팀과 인터페이스 조율, 코드 리뷰 |

### 5.2 기술 스택
| 분류 | 도구 |
|------|------|
| 언어 | Python 3.12.3 |
| 웹 프레임워크 | **React** |
| LLM 연동 | `openai`, `langchain` (R/G 팀 파이프라인 직접 import) |
| Ollama 연동 (스트레치) | `langchain-ollama`, `requests` |
| 환경 변수 | `python-dotenv` |
| 로깅 | `loguru` 또는 표준 `logging` |

### 5.3 GCP VM 활용 (Ollama 서빙용)
- VM 사양: g2-standard-4, L4 GPU, 16GB RAM, 100GB Disk
- Ollama 설치: `curl -fsSL https://ollama.com/install.sh | sh`
- 모델 추천: `llama3:8b`, `gemma2:9b`, `qwen2.5:7b` (한국어 양호)
- 포트포워딩: GCP 방화벽 11434 포트 오픈 + SSH 터널
- LangChain 연동: `from langchain_ollama import ChatOllama`

---

## 6. 위험 관리 (Risk Management)

| # | 리스크 | 발생가능성 | 영향도 | 완화 전략 |
|---|--------|----------|--------|----------|
| R1 | **R/G 팀 파이프라인 지연** | 高 | 高 | **Mock 인터페이스로 우선 개발** (Sprint 0~1) |
| R2 | **인터페이스 변경** (R/G 팀이 함수 시그니처 바꿈) | 中 | 高 | M1(06/22) 시점에 인터페이스 락(lock), 변경 시 전민재 PM 협의 |
| R3 | **1인 개발 → 막힘 시 진행 정체** | 高 | 高 | 30분 룰: 막히면 무조건 Discord 호출 |
| R4 | **React 세션 상태 버그** (새로고침 시 대화 사라짐) | 高 | 中 | `st.session_state` 패턴 학습, 필요시 파일 저장 |
| R5 | **Ollama 연동 시간 부족** | 高 | 中 | M2 이후 시도, 안 되면 백로그 (OpenAI만으로 데모 가능) |
| R6 | **GCP VM 포트포워딩 어려움** | 中 | 中 | SSH 터널 가이드 사전 학습, 안 되면 Ollama는 VM 내부에서만 |
| R7 | **응답 시간 너무 김** (>15초) | 中 | 中 | 로딩 인디케이터 추가, R/G 팀에 Top-k 조정 요청 |
| R8 | **에러 핸들링 누락 → 데모 망함** | 中 | 高 | 모든 외부 호출에 try-except + 사용자 친화 메시지 |
| R9 | **시간 부족으로 UX 개선 못 함** | 高 | 中 | 가이드의 "미구현 항목" 명확 — 다크모드/마크다운/모바일은 처음부터 백로그 |
| R10 | **모듈화 과욕** (시간 부족한데 리팩토링) | 中 | 高 | "동작 먼저, 리팩토링 나중" 원칙, M1까지는 기능 우선 |

---

## 7. 품질 보증 (Quality Assurance)

### 7.1 기능 품질 기준
| 항목 | 목표 | 측정 방법 |
|------|------|----------|
| 챗봇 UI 동작 | 메시지 입출력 100% 정상 | 수동 테스트 30회 |
| RAG 응답 성공률 | ≥ 95% (네트워크 정상 시) | 시나리오 60건 테스트 |
| 평균 응답 시간 | ≤ 10초 | 자동 측정 로그 |
| 에러 발생 시 UX | 빈 화면/크래시 0건 | 의도적 에러 주입 테스트 |
| 대화 맥락 유지 | 후속 질의 정상 처리 | 시나리오 15건 (후속 유형) |

### 7.2 통합 테스트 시나리오 (M2 직전 필수 수행)

> 골든 데이터셋 60건을 데이터팀에서 받아, 4가지 유형별 샘플 테스트:

1. **단일문서 추출 (5건)**: "국민연금공단 이러닝 사업 예산은?" → 정확한 금액 답변
2. **다중문서 종합 (5건)**: "교육 관련 사업 발주 기관 모두 알려줘" → 여러 기관 나열
3. **후속 질의 (5건)**: 선행 질문 후 "그럼 모니터링 요청사항은?" → 맥락 유지 확인
4. **답변 거부 (5건)**: 문서 외 정보 질문 → "모른다" 답변 확인
5. **에러 시나리오 (5건)**: 빈 입력, 매우 긴 입력, 네트워크 끊김 → 친화적 에러

### 7.3 코드 품질 기준
- API 키는 `.env` (절대 하드코딩 금지, NDA 위반 가능성)
- 모든 외부 호출(LLM, R/G 파이프라인)에 try-except 필수
- 사용자에게 노출되는 에러 메시지는 친화적 톤
- 로깅: 질의/응답/응답시간/에러 모두 기록

---

## 8. 인접팀과의 인터페이스 (Interface)

### 8.1 R/G 팀 → 서비스 팀 (수신)
**M1(06/22) 시점에 인터페이스 락**
```python
from rag.pipeline import get_ai_response

result = get_ai_response(
    query="국민연금공단 이러닝 사업 예산은?",
    history=[
        {"role": "user", "content": "이전 질문"},
        {"role": "assistant", "content": "이전 답변"}
    ]
)
# result 구조 (R/G 팀과 합의)
{
    "answer": "5억 4천만원입니다.",
    "sources": [{"doc_id": "DOC_001", "page": 12, "score": 0.92}],
    "retrieved_chunks": ["본 사업의 총 예산은..."],
    "elapsed_sec": 6.2,
    "tokens_used": 850
}
```

### 8.2 서비스 팀 내부 모듈화 구조 (가이드 인용)
```
[React UI] 
    ↓
[service/pipeline.get_ai_response()]   ← 백엔드 진입점
    ↓
[service/llm_client]  [service/prompt_templates]  [service/memory]
    ↓                                                ↓
[R/G 팀 RAG]                                  [대화 이력 관리]
    ↓
[FAISS Vector DB / OpenAI gpt-5-mini]
```

### 8.3 전민재 PM에게 보고할 사항
- M1(06/22): UI 데모 영상 1분
- M2(06/25): 통합 데모 영상 3분 (시나리오 4종)
- 06/26: 최종 데모 + 발표 슬라이드

---

## 9. 협업 일지 및 데일리 스크럼

### 9.1 협업일지 작성
- 매일 시작 + 마무리 2회 작성 (1인 개발 하더라도 업무 일지 작성 및 공유 중요)
- 필수 항목: 오늘 할 일 / 어제 한 일 / Blocker / 팀 기여 / 학습

### 9.2 데일리 스크럼
- **시간**: 매일 10:00 KST, Discord 음성, 10분
- 1인 개발 업무하더라도 R/G/데이터 팀과 매일 소통 필수
- 특히 R/G 팀과의 **인터페이스 변경 사항을 가장 먼저 인지해야 함**

### 9.3 KPT 회고
- M1(06/22), M2(06/25) 2회
- 전민재 PM과 1:1 가능 (혼자라도 본인 KPT는 작성)

---

## 10. 부록 (Appendix)

### A. 디렉토리 구조(데이터 전처리 + 서비스 개발 챗봇 + RAG 시스템 합친 디렉토리 구조)
```
chatbot/
├── backend/    ← FastAPI (웹/앱 공통)
|   ├── data/                 # ⚠️ .gitignore (NDA)
│   |   ├── raw/              # 원본 HWP, PDF
│   |   ├── metadata/         # data_list.csv
│   |   └── processed/        # 가공 데이터 (청크 JSON 등)
|   |
|   ├── golden/ (data 폴더와 합칠지 따로 둘지 논의 필요)
│   |   └── golden_dataset.json
|   |
│   ├── retrieval/
│   │   ├── naive_retriever.py
│   │   ├── agentic_retriever.py   # LangGraph 기반
│   │   ├── embedder.py
│   │   ├── vector_store.py
│   │   └── loader.py
|   |
│   ├── generation/
│   │   ├── generator.py
│   │   ├── llm_client.py
│   │   ├── memory.py
│   │   └── prompts/
│   │       ├── system_v1.txt
│   │       └── system_v2.txt
|   |
│   ├── evaluation/
│   │   ├── eval_retrieval.py     # Hit Rate, MRR, Context P/R
│   │   ├── eval_generation.py    # Faithfulness, Answer Relevance
│   │   ├── eval_ragas.py
│   │   ├── eval_text.py          # ROUGE, BERTScore, BLEU
│   │   └── llm_judge.py
│   └── pipeline.py               # run_rag_pipeline(), get_ai_response()
|
├── frontend/   ← React 웹
├── mobile/     ← React Native 앱
|
├── utils/
│   └── config.py               # retriever_type, llm_model, top_k 등
|
├── notebooks/
│   ├── 01_EDA.ipynb
│   ├── 02_loading_test.ipynb
│   ├── 03_chunking_stats.ipynb
│   ├── 04_mapping_validation.ipynb
│   ├── 05_naive_rag_test.ipynb
│   ├── 06_agentic_rag_test.ipynb
│   └── 07_evaluation.ipynb
|
├── reports/
|   ├── processing_report.md
|   ├── chunking_analysis.md
|   └── eval_report.md
|
├── requirements.txt
├── .gitignore
└── README.md
```

### B. 실행 명령어 (README용)
```bash
# 1. 환경 셋업
pip install -r requirements.txt

# 2. 환경변수 설정
cp .env.example .env
# .env 파일 편집 → OPENAI_API_KEY=...

# 3. 실행
python chatbot/backend/main.py
```

### C. 일일 체크리스트 템플릿
```markdown
## 📅 YYYY-MM-DD 데일리 체크리스트 (1인 개발 전용)

### ☀️ 아침 (10:00 스크럼 전)
- [ ] R/G 팀 인터페이스 변경사항 확인 (Discord/Notion)
- [ ] 오늘 할 일 3가지 + 우선순위
- [ ] 30분 룰 다짐 (막히면 30분 안에 도움 요청)

### 🌤️ 점심 (13:00)
- [ ] 오전 결과 GitHub push
- [ ] Blocker 있으면 즉시 전민재 PM/Discord

### 🌙 저녁 (17:00)
- [ ] 코드 커밋/푸시
- [ ] 협업일지 작성 (혼자라도 매일!)
- [ ] 내일 R/G 팀 의존 작업 미리 알림
```

### D. 미구현 항목 (가이드 명시 — 본 스프린트 X)
- 로그인 / JWT 인증
- 사용자별 대화 저장 (PostgreSQL 등 별도 DB)
- DB 구축 → 로그인 → 대화 저장 순서로 **후속 프로젝트에서 진행**
- 다크모드, 마크다운 렌더링, 모바일 반응형 (주제 확정 전 기능)
- 파인튜닝 모델 교체 (주제 확정 후 기능)

### E. 보안 및 NDA 주의사항
- ⚠️ OpenAI API 키 절대 GitHub commit 금지 (`.gitignore` 검증)
- ⚠️ SSH 키, GCP 서비스 계정 키 보안 주의
- ⚠️ GenAI(ChatGPT/Claude 등)에 API 키 포함 대화 금지
- ⚠️ 원본 RFP 데이터 외부 공유 금지 — 데모 영상 녹화 시 화면에 RFP 원문 노출 주의

---

## 11. 변경 이력

| 버전 | 일자 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| v1.0 | 2026-06-18 | 최초 작성 | 전민재 PM |

---

> 🎯 **참고 사항**:
> "동작하는 mock이 동작하지 않는 완벽한 코드보다 가치 있다."  
> M1까지는 **mock으로라도** 풀스택이 돌아가야 하고, M2부터 실제 통합.  
> 막힘은 부끄러운 게 아니라 **공유하는 것이 협업의 본질**입니다.  
> "구현하고 싶은 기능이 많더라도 8일 안에 끝낼 수 없다면 과감히 백로그로." (가이드 인용)
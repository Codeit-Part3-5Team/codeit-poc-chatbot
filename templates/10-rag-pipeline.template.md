# 🔍 Retrieval / Generation 담당 전용 소프트웨어 개발 계획서 (SDP)
## B2G RFP RAG 시스템 — Retrieval & Generation Team

> **프로젝트명**: 입찰메이트 사내 RAG 시스템 구축 (B2G 입찰지원 컨설팅)  
> **문서 버전**: v1.0  
> **작성일**: 2026-06-18  
> **작성자**: 전민재 PM
> **대상 팀**: Retrieval / Generation 담당 (김재헌A님, 노수민님)
> **프로젝트 기간**: 2026-06-19(금) ~ 2026-06-26(금) — 총 8일

---

## 1. 프로젝트 개요 (Project Overview)

### 1.1 프로젝트 배경
- 가상의 B2G 입찰지원 컨설팅 스타트업 **'입찰메이트'**의 사내 RAG 시스템 구축.
- 100건의 RFP(HWP/PDF)에 대해 컨설턴트가 자연어 질의로 정보를 빠르게 추출.
- **Retrieval/Generation 팀**은 RAG 시스템의 **두뇌(Brain)** — 검색 정확도와 답변 품질이 곧 서비스 가치.

### 1.2 R/G 팀의 미션
> **"데이터 팀이 가공한 청크를 받아, 사용자 질문에 가장 적합한 컨텍스트를 검색하고, 환각 없이 근거 기반 답변을 생성한다."**

### 1.3 핵심 설계 원칙 (Hybrid Architecture)
> **Naive RAG ↔ Agentic RAG 둘 다 사용 가능한 하이브리드 구조**  
> `config.yaml`의 `retriever_type` 값에 따라 분기:
> - `retriever_type: "naive_rag"` → `naive_retriever` 모듈 호출
> - `retriever_type: "agentic_rag"` → `agentic_retriever` 모듈 호출 (LangChain/LangGraph)

### 1.4 최종 목표 (Definition of Done)
| # | 산출물 | 완료 기준 |
|---|--------|----------|
| 1 | 통합 RAG Pipeline 함수 (`run_rag_pipeline()`) | config.yaml 토글로 naive/agentic 분기 동작 |
| 2 | Naive Retriever | similarity_search + 메타데이터 필터링 동작 |
| 3 | Agentic Retriever (LangGraph) | Multi-Query 또는 Re-Ranking 1종 이상 적용 |
| 4 | Generator 모듈 | gpt-5-mini 연동 + 대화 히스토리 반영 |
| 5 | 평가 리포트 | 골든셋 60건으로 Hit Rate, MRR, Faithfulness, RAGAS-Score 산출 |
| 6 | LLM Judge 2단계 로직 | 1단계 Ollama → 2단계 Cloud LLM 비교 답변 |
| 7 | 서비스팀 인터페이스 | `get_ai_response(query, history)` 함수 export |

---

## 2. 범위 정의 (Scope)

### 2.1 ✅ 포함 범위 (In-Scope)

**Retrieval 파트**
- 임베딩 모델 연동: `text-embedding-3-small` (OpenAI) — 1순위
- Vector DB: **FAISS** (1순위, 로컬 + 빠른 실험) / Chroma (백업)
- 베이스라인: `similarity_search` (Top-k)
- 심화 1: `similarity_search_with_score`, `max_marginal_relevance_search` (MMR)
- 심화 2: 메타데이터 필터링 (발주기관/사업명 — 유사 문자열 대응)
- 심화 3: Multi-Query 또는 Re-Ranking 중 **1종 선택 구현** (시간 부족 시 단일 선택)
- Hybrid Search (BM25 + Vector)는 **백로그** (선택)

**Generation 파트**
- LLM: **gpt-5-mini** (1순위), gpt-5-nano (비용 절약 케이스)
- 프롬프트 템플릿 시스템 (System / Human / Tool 메시지 분리)
- 생성 옵션: temperature, top_p, max_tokens 최적화
- 대화 히스토리(Memory) 유지: 최근 N턴 + 요약 방식
- 답변 거부 로직: "문서에 없음" 명시적 처리
- LLM Judge 2단계: GCP Ollama 답변 ↔ Cloud LLM 답변 비교

**평가 (Evaluation)**
- Retrieval 지표: **Hit Rate**, **MRR**, Context Recall, Context Precision
- Generation 지표: **Faithfulness**, **Answer Relevance**
- 종합 지표: **RAGAS-Score**
- 텍스트 품질: ROUGE-1, ROUGE-L, BERTScore-F1, BLEU
- 골든 데이터셋 60건(데이터팀 제공) 활용

### 2.2 ❌ 미포함 범위 (Out-of-Scope) — 백로그로 이관
- Hybrid Search (BM25 + Dense) 완전 구현 — 시간 부족 시 제외
- 자체 임베딩 모델 파인튜닝
- Self-RAG, Adaptive RAG 등 최신 논문 기법
- 멀티모달 RAG (이미지 처리)
- 프롬프트 인젝션/탈옥 방어 (보안 취약점 개선) — 최종 1일 가능 시 도전
- 도메인 전문가 튜링 테스트 (가이드에서 "팀 프로젝트 진행 제외"로 명시됨)

> **⚠️ Scope Creep 방지**: "이 기법도 적용하면 좋겠다"는 의견은 GitHub Issue에 백로그로 등록. 본 스프린트 내 추가 작업은 전민재 PM 승인 필수.

---

## 3. 방법론 (Methodology)

### 3.1 개발 방법론: 애자일 + 일단위 스프린트
- **2단계 미니 스프린트** 구조 (Sprint 1: 베이스라인 / Sprint 2: 고도화)
- 매일 **데일리 스크럼 10분** (10:00 KST 고정)
  - 어제 한 일 / 오늘 할 일 / 막히는 부분(Blocker)
- 미니 스프린트 종료 시 **KPT 회고** (Keep / Problem / Try)

### 3.2 협업 규칙
| 항목 | 규칙 |
|------|------|
| 브랜치 | `feat/retrieve-*`, `feat/generate-*`, `feat/evaluate-*` |
| PR 리뷰 | R/G 팀원 상호 리뷰 → 전민재 PM 최종 승인 |
| 일일 머지 | 최소 2일 1회 `dev` 브랜치 머지 |
| 실험 노트 | Notion에 프롬프트/하이퍼파라미터 실험 결과 기록 |
| 코드 컨벤션 | `[Retrieve] feat: MMR 추가`, `[Generate] fix: 프롬프트 길이 제한` |

### 3.3 기술 스택 결정 (확정)
| 분류 | 선택 | 이유 |
|------|------|------|
| 시나리오 | **B (API 우선) → A (Ollama 추가)** | 가이드 권장, 8일 안에 베이스라인 확보 우선 |
| LLM | gpt-5-mini | API 한도 $20 내 충분, 한국어 성능 우수 |
| Embedding | text-embedding-3-small | 비용 효율, 다국어 지원 |
| Vector DB | FAISS | 로컬 실행, 빠른 실험, 의존성 적음 |
| 프레임워크 | LangChain (필수) + LangGraph (Agentic만) | 가이드 명시 |
| 평가 | RAGAS, sklearn metrics | 표준 RAG 평가 |

---

## 4. 일정 및 마일스톤 (Schedule & Milestones)

### 4.1 전체 타임라인 (8일)

| 일자 | 요일 | 미니 스프린트 | 주요 마일스톤 |
|------|------|---------------|--------------|
| 06/19 | 금 | **Sprint 0**: 셋업 | 환경 구축 + LangChain 학습 |
| 06/20 | 토 | **Sprint 1**: Naive RAG | Loader/Embedding/Retriever 베이스라인 |
| 06/21 | 일 | **Sprint 1** | Generator 베이스라인 + 엔드투엔드 동작 |
| 06/22 | 월 | **Sprint 1** | ⭐ **M1: Naive RAG E2E 완성** + KPT 회고 |
| 06/23 | 화 | **Sprint 2**: Agentic + 평가 | Agentic Retriever + 평가 파이프라인 |
| 06/24 | 수 | **Sprint 2** | LLM Judge 2단계 + Memory + 골든셋 실험 |
| 06/25 | 목 | **Sprint 2** | ⭐ **M2: 성능 평가 완료** + KPT 회고 |
| 06/26 | 금 | **Wrap-up** | 문서화, 발표자료 모델 파트, 보고서 통합 |

### 4.2 WBS (Work Breakdown Structure) — R/G 팀 2인

#### 🎯 역할 분담
- **Engineer A (Retrieval Lead)**: 문서 로더 연동, 임베딩, Vector DB, Retriever 구현, Retrieval 평가
- **Engineer B (Generation Lead)**: LLM 연동, 프롬프트, Memory, Generation 평가, LLM Judge

> ⚠️ **명확한 인터페이스가 핵심**: A와 B는 `Retriever → context → Generator` 인터페이스만 합의하면 병렬 작업 가능.

#### 🔹 Sprint 0: 셋업 (06/19, 1일)
| 담당 | 작업 | 산출물 |
|------|------|--------|
| 공동 | OpenAI API 키 환경변수 설정, `.env` 셋업 | `.env.example` |
| 공동 | `langchain`, `langchain-openai`, `faiss-cpu`, `ragas` 설치 | `requirements.txt` |
| Eng A | LangChain Retriever 공식 튜토리얼 1회 수행 | 학습 노트 |
| Eng B | LangChain LCEL + Prompt Template 튜토리얼 1회 수행 | 학습 노트 |
| 공동 | `config.yaml` 스키마 합의 (retriever_type, llm_model, top_k 등) | `config.yaml` |
| 공동 | 데이터팀의 청크 출력 스키마 인터페이스 합의 | 인터페이스 문서 |

#### 🔹 Sprint 1: Naive RAG 베이스라인 (06/20 ~ 06/22, 3일)

**Engineer A — Retrieval 파트**
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| 청크 JSON 로딩 → Document 객체 변환 | 0.5일 | `loader.py` |
| OpenAI Embedding 연동 + 캐싱 | 0.5일 | `embedder.py` |
| FAISS 인덱스 생성 + 저장/로드 | 0.5일 | `vector_store.py` |
| `naive_retriever.py`: similarity_search Top-k=5 | 0.5일 | `naive_retriever.py` |
| 메타데이터 필터 (발주기관, 사업명) 기본 구현 | 0.5일 | 필터 함수 |
| Retriever 단위 테스트 (샘플 질의 10건) | 0.5일 | 테스트 노트북 |

**Engineer B — Generation 파트**
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| gpt-5-mini ChatOpenAI 연동 + 옵션 튜닝 | 0.5일 | `llm_client.py` |
| 시스템 프롬프트 v1 작성 (RFP 도메인 + 거부 규칙) | 0.5일 | `prompts/system_v1.txt` |
| `generator.py`: context + question → answer | 0.5일 | `generator.py` |
| `run_rag_pipeline(query, config)` 함수 | 0.5일 | `pipeline.py` |
| `config.yaml` 분기 로직 (naive_rag/agentic_rag) | 0.5일 | 분기 코드 |
| 엔드투엔드 동작 검증 (샘플 5건) | 0.5일 | 검증 로그 |

#### 🔹 Sprint 2: Agentic RAG + 평가 (06/23 ~ 06/25, 3일)

**Engineer A — Agentic Retriever + 평가**
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| MMR (`max_marginal_relevance_search`) 추가 | 0.5일 | `retriever_mmr.py` |
| Multi-Query Retrieval (LangChain MultiQueryRetriever) | 1일 | `agentic_retriever.py` |
| ⚠️ Re-Ranking은 시간 부족 시 백로그 | (조건부) | - |
| Retrieval 평가: Hit Rate, MRR, Context Recall/Precision | 1일 | `eval_retrieval.py` |
| 골든셋 60건으로 Naive vs Agentic 비교 실험 | 0.5일 | 비교 리포트 |

**Engineer B — Generator 고도화 + LLM Judge**
| 작업 | 예상 공수 | 산출물 |
|------|----------|--------|
| ConversationBufferWindowMemory (최근 N턴 유지) | 0.5일 | Memory 통합 |
| 프롬프트 v2: 후속 질의 맥락 처리 | 0.5일 | `prompts/system_v2.txt` |
| Generation 평가: Faithfulness, Answer Relevance (RAGAS) | 1일 | `eval_generation.py` |
| RAGAS-Score 종합 산출 | 0.5일 | `eval_ragas.py` |
| LLM Judge 2단계: Ollama 답변 → Cloud LLM 검증 | 0.5일 | `llm_judge.py` |
| (선택) ROUGE/BERTScore/BLEU 계산 | (선택) | `eval_text.py` |

#### 🔹 Wrap-up: 문서화 + 발표 (06/26, 1일)
| 담당 | 작업 | 산출물 |
|------|------|--------|
| Eng A | Retrieval README + 청킹×Retriever 매트릭스 표 | `retrieval/README.md` |
| Eng B | Generator README + 프롬프트 진화 히스토리 | `generation/README.md` |
| 공동 | 발표 자료 모델 파트 5~6 슬라이드 작성 | 슬라이드 |
| 공동 | 보고서 모델/평가 파트 통합 | 보고서 일부 |

### 4.3 일일 작업량 가이드 (초급 엔지니어 현실 기준)
- 하루 순수 코딩 시간 **4~5시간** 기준
- LangChain은 변경이 잦아 **공식 문서 확인을 우선** (블로그 코드는 deprecated 가능성↑)
- 매일 16:00까지 코드 작성 → 16:00~17:00 리뷰/머지 → 17:00 데일리 종료
- **LangGraph는 가장 어려운 부분** — Sprint 2 진입 전에 공식 튜토리얼 1회 학습 필수

---

## 5. 자원 및 역할 (Resources & Roles)

### 5.1 인력 (Resources)

| 역할 | 담당자 | R&R |
|------|--------|-----|
| **Engineer A (Retrieval Lead)** | TBD | Loader, Embedding, Vector DB, Retriever, Retrieval 평가 |
| **Engineer B (Generation Lead)** | TBD | LLM, Prompt, Memory, Generation 평가, LLM Judge |
| **전민재 PM (지원)** | 팀장 | 인접팀(데이터/서비스) 인터페이스 조율, 평가 총괄 |

### 5.2 기술 스택 상세

| 분류 | 도구 / 라이브러리 |
|------|------------------|
| 언어 | Python 3.12.3 |
| 핵심 | `langchain`, `langchain-openai`, `langchain-community`, `langgraph` |
| LLM | OpenAI `gpt-5-mini` (1순위), `gpt-5-nano` (백업) |
| Embedding | OpenAI `text-embedding-3-small` |
| Vector DB | `faiss-cpu` (1순위), `chromadb` (백업) |
| 평가 | `ragas`, `scikit-learn`, `rouge-score`, `bert-score` |
| 실험 추적 | Notion 표 (또는 간단한 `experiments.csv`) |
| (선택) Ollama | LLM Judge 1단계용 — Sprint 2 후반 |

### 5.3 API 사용량 관리 (⚠️ 중요)
- 팀 OpenAI 할당량: **$20** (초과 시 키 disabled)
- **개발 중 임베딩 캐싱 필수** — 같은 청크 반복 임베딩 절대 금지
- 평가 실험은 **소규모 우선** (60건 골든셋 → 처음엔 10건 샘플 검증 → 전체 확장)
- 디스코드 팀 채널에서 `!usage` 명령으로 사용량 수시 확인 (1분 1회)

---

## 6. 위험 관리 (Risk Management)

| # | 리스크 | 발생가능성 | 영향도 | 완화 전략 |
|---|--------|----------|--------|----------|
| R1 | **LangChain 버전 호환성 이슈** (deprecated API) | 高 | 高 | 공식 docs 기준 작성, `requirements.txt` 버전 고정 |
| R2 | **LangGraph 학습 곡선** | 高 | 高 | Sprint 1에 1시간 페어 학습 세션, 어려우면 LangChain만으로도 Agentic 구현 |
| R3 | **OpenAI API 한도 초과** ($20) | 中 | 高 | 임베딩 캐싱, 평가 샘플 단계적 확대, `!usage` 모니터링 |
| R4 | **임베딩-청크 차원 불일치** | 中 | 中 | text-embedding-3-small (1536차원) 고정, 인덱스 재생성 절차 문서화 |
| R5 | **메타데이터 필터링 부정확** (오타/약칭) | 高 | 中 | 데이터팀의 alias 사전 활용, fuzzy matching (`rapidfuzz`) 적용 |
| R6 | **RAGAS 평가 시 API 호출 폭증** | 高 | 高 | 평가는 **샘플 20건 → 60건 전체** 단계적, GPT-5-mini만 사용 |
| R7 | **데이터팀 청크 지연** | 中 | 高 | M1까지 더미 청크 10건으로 파이프라인 선구축 |
| R8 | **프롬프트 길이 초과 (context window)** | 中 | 中 | Top-k=3~5로 제한, 청크 길이 검증 |
| R9 | **답변 환각(hallucination)** | 高 | 高 | 시스템 프롬프트에 "문서 외 정보는 모른다고 답변" 명시, Faithfulness 모니터링 |
| R10 | **8일 안에 Agentic까지 완성 어려움** | 高 | 中 | M1까지 Naive 무조건 완성, Agentic은 M1 이후 도전 |
| R11 | **서비스팀과 인터페이스 불일치** | 中 | 高 | M1 시점에 `get_ai_response(query, history)` 시그니처 확정 |

---

## 7. 품질 보증 (Quality Assurance)

### 7.1 성능 평가 지표 및 목표

> **골든 데이터셋이란?** AI 모델의 성능을 평가하고 검증하기 위해 사람이 직접 검증하여 신뢰성을 확보한 고품질의 표준(Reference) Q&A 데이터셋. 데이터팀에서 60건 제공.

#### Retrieval 평가 지표
| 지표 | 정의 | 목표치 (베이스라인) | 목표치 (Agentic) |
|------|------|---------------------|------------------|
| **Hit Rate @5** | 상위 5개 결과에 정답 문서 포함 비율 | ≥ 0.70 | ≥ 0.85 |
| **MRR (Mean Reciprocal Rank)** | 정답의 평균 역순위 | ≥ 0.50 | ≥ 0.65 |
| **Context Recall** | 정답 컨텍스트가 검색에 포함된 비율 | ≥ 0.65 | ≥ 0.80 |
| **Context Precision** | 검색 결과의 관련성 비율 | ≥ 0.55 | ≥ 0.70 |

#### Generation 평가 지표
| 지표 | 정의 | 목표치 |
|------|------|--------|
| **Faithfulness** | 답변이 컨텍스트에 충실한 정도 | ≥ 0.75 |
| **Answer Relevance** | 답변이 질문에 적절한 정도 | ≥ 0.75 |
| **RAGAS-Score** | 종합 점수 | ≥ 0.70 |

#### 응답 속도
- 평균 응답 시간 ≤ **8초** (Top-5 검색 + 생성 포함)

### 7.2 평가 시나리오 (4가지 유형)
1. **단일문서 추출** (20건) — 예: "국민연금공단 이러닝시스템 사업 예산?"
2. **다중문서 종합** (15건) — 예: "교육 관련 사업 발주 기관 모두 알려줘"
3. **후속 질의** (15건) — 예: (선행 후) "콘텐츠 개발 관리 요구사항은?"
4. **답변 거부** (10건) — 예: "이 사업 낙찰업체는?" (문서 외 정보)

### 7.3 LLM Judge 2단계 로직
```
[사용자 질문]
   ↓
[1단계] GCP 서버 Ollama 모델 답변 생성 (e.g., Llama 3 8B)
   ↓ 
[2단계] OpenAI gpt-5-mini가 다음을 판정:
   - Ollama 답변 vs Ground Truth 비교
   - 점수(1~5) + 사유 출력
   ↓
[결과 저장] → 평가 로그
```

### 7.4 코드 품질 기준
- 모든 함수에 docstring (목적/입력/출력/예외)
- 타입 힌트 권장 (`def retrieve(query: str, k: int = 5) -> List[Document]:`)
- 환경변수: API 키는 `.env`로만 (`.gitignore` 필수)
- 로깅: 검색 결과/생성 시간/토큰 사용량 모두 기록

---

## 8. 인접팀과의 인터페이스 (Interface)

### 8.1 데이터 팀 → R/G 팀 (입력)
**Sprint 1 시작 시 (06/22) 수신**
```json
{
  "doc_id": "DOC_001",
  "doc_metadata": {
    "agency": "국민연금공단",
    "agency_aliases": ["국민연금", "NPS"],
    "project_name": "이러닝시스템 고도화 사업",
    "budget_krw": 540000000
  },
  "chunks": [
    {"chunk_id": "DOC_001_C001", "text": "...", "section": "1. 사업개요", "page": 1}
  ]
}
```

### 8.2 R/G 팀 → 서비스 팀 (출력)
**서비스팀이 호출할 메인 인터페이스 — M1 시점 확정**
```python
def get_ai_response(
    query: str,
    history: list[dict] = None,   # [{"role": "user|assistant", "content": "..."}]
    config: dict = None            # config.yaml 로드된 dict (선택)
) -> dict:
    """
    Returns:
        {
            "answer": str,                     # 최종 답변
            "sources": list[dict],             # 참조 문서 [{doc_id, page, score}]
            "retrieved_chunks": list[str],     # 검색된 청크 텍스트
            "elapsed_sec": float,              # 응답 시간
            "tokens_used": int                 # 토큰 사용량
        }
    """
```

> ⚠️ **이 인터페이스는 M1(06/22) 종료 시점에 확정**. 이후 변경 시 전민재 PM 협의 필수.

### 8.3 R/G 팀 → 전민재 PM (성능 리포트)
M2(06/25) 종료 시점에 다음 산출물 전달:
- 골든셋 60건 평가 결과 표
- Naive vs Agentic 비교 차트
- 실패 케이스 분석 (Bottom 5 질문)

---

## 9. 협업 일지 및 데일리 스크럼

### 9.1 협업일지 작성 (개인 필수)
매일 작성, 시작 + 마무리 2회. 필수 항목:
- 오늘 할 일 / 어제 한 일 / Blocker
- 팀 기여 내용 / 학습 / 감정 소감

### 9.2 데일리 스크럼 진행 방식
- **시간**: 매일 10:00 KST, 10분 엄수, Discord 음성
- **순서**: Eng A → Eng B → 전민재 PM 코멘트
- **다른 사람이 무엇을 하고 있는지 파악**하는 것도 중요

### 9.3 미니 스프린트 KPT 회고
- **M1 종료 (06/22)**, **M2 종료 (06/25)** 2회
- 30분 내외, Notion 기록

---

## 10. 부록 (Appendix)

### A. 디렉토리 구조(서비스 개발 챗봇 + RAG 시스템 합친 디렉토리 구조)
```
chatbot/
├── backend/    ← FastAPI (웹/앱 공통)
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
│   ├── 01_naive_rag_test.ipynb
│   ├── 02_agentic_rag_test.ipynb
│   └── 03_evaluation.ipynb
|
└── reports/
    └── eval_report.md
```

### B. config.yaml 예시
```yaml
retriever_type: "naive_rag"     # or "agentic_rag"
top_k: 5
embedding_model: "text-embedding-3-small"
llm_model: "gpt-5-mini"
temperature: 0.2
max_tokens: 1024

agentic:
  multi_query: true
  rerank: false                  # 시간 부족 시 false

memory:
  window_size: 5                 # 최근 5턴 유지

evaluation:
  golden_dataset_path: "./golden/golden_dataset.json"
  sample_size: 60
```

### C. 일일 체크리스트 템플릿
```markdown
## 📅 YYYY-MM-DD 데일리 체크리스트 (개인용)

### ☀️ 아침 (10:00 스크럼 전)
- [ ] 어제 PR 리뷰 상태 확인
- [ ] 오늘 할 일 3가지 (우선순위 순)
- [ ] OpenAI API `!usage` 확인

### 🌤️ 점심 (13:00)
- [ ] 오전 진척률 자가 점검
- [ ] Blocker 있으면 Discord 즉시 공유

### 🌙 저녁 (17:00 마감)
- [ ] 코드 커밋/PR 완료
- [ ] 협업일지 작성
- [ ] 내일 할 일 메모
```

### D. 평가 시 참고 질문 예시 (가이드 출처)
1. "국민연금공단이 발주한 이러닝시스템 관련 사업 요구사항을 정리해 줘."
   - 후속: "콘텐츠 개발 관리 요구 사항에 대해서 더 자세히 알려 줘."
2. "기초과학연구원 극저온시스템 사업 요구에서 AI 기반 예측에 대한 요구사항이 있나?"
3. "고려대학교 차세대 포털 시스템 사업이랑 광주과학기술원의 학사 시스템 기능개선 사업을 비교해 줘."

### E. 보안 및 NDA 주의사항
- API 키, SSH 키는 `.gitignore` 필수
- 원본 RFP 데이터 GitHub 업로드 금지 (NDA)
- GenAI(ChatGPT 등)에 API 키 포함 대화 금지
- 평가 결과/코드 등 **2차 가공물만** 외부 공유 가능

---

## 11. 변경 이력

| 버전 | 일자 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| v1.0 | 2026-06-18 | 최초 작성 | 전민재 PM |

---
 
> M1 종료(06/22) 시점에 Agentic 범위 재조정, M2 종료(06/25) 시점에 평가 결과 기반 백로그 최종 결정.  
> "구현하고 싶은 기능이 많더라도 8일 안에 끝낼 수 없다면 과감히 백로그로." (가이드 인용)
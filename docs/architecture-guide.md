# MiroFish 아키텍처 가이드

> 멀티 에이전트 AI 예측 엔진의 전체 구조와 데이터 흐름 정리

## 1. 전체 흐름도

```
사용자 입력 (PDF/MD/TXT + 자연어 요구사항)
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Step 1: 온톨로지 생성                                │
│  POST /api/graph/ontology/generate                    │
│  파일 업로드 → LLM 분석 → 엔터티/관계 타입 정의       │
│  (엔터티 10개, 관계 6-10개)                           │
└──────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Step 2: 그래프 구축 (비동기)                         │
│  POST /api/graph/build                                │
│  텍스트 청크 분할 → Zep Cloud에 에피소드 추가         │
│  → GraphRAG 자동 처리 → 지식 그래프 완성              │
└──────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Step 3: 시뮬레이션 환경 준비                         │
│  POST /api/simulation/prepare                         │
│  3a: Zep 엔터티 추출 (ZepEntityReader)                │
│  3b: LLM 기반 Agent Profile 생성 (병렬 3-5개)        │
│       → Twitter CSV + Reddit JSON 출력                │
│  3c: 시뮬레이션 설정 자동 생성                        │
│       → 시간, 활성도, 발화 빈도 등                    │
└──────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Step 4: OASIS 시뮬레이션 실행                        │
│  외부 스크립트 (Python subprocess)                     │
│  run_twitter_simulation.py / run_reddit_simulation.py │
│  → 에이전트 상호작용 (라운드 기반)                    │
│  → 포스트, 좋아요, 리포스트, 댓글 등 액션 생성        │
│  → 시뮬레이션 중 Zep 메모리 실시간 업데이트           │
└──────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────┐
│  Step 5: 보고서 생성 + 심화 상호작용                  │
│  POST /api/report/generate                            │
│  ReportAgent (도구 활용):                             │
│    - Deep Insight: Zep 그래프 검색                    │
│    - Agent Interview: 시뮬레이션 에이전트 인터뷰      │
│    - Panorama Search: 전체 기억 검색                  │
│    - Quick Search: 빠른 검색                          │
│  → 섹션별 마크다운 보고서 생성                        │
│  POST /api/report/chat → 에이전트와 대화              │
└──────────────────────────────────────────────────────┘
```

## 2. 기술 스택

| 계층 | 기술 | 용도 |
|------|------|------|
| **프론트엔드** | Vue 3 + Vite | SPA UI |
| **백엔드** | Python 3.11 + Flask | REST API |
| **LLM** | OpenAI SDK 호환 | 온톨로지/프로필/보고서 생성 |
| **메모리 그래프** | Zep Cloud | 지식 그래프 + GraphRAG |
| **시뮬레이션** | OASIS (CAMEL-AI) | 멀티 에이전트 시뮬레이션 |
| **패키지** | uv (Python), npm (Node) | 의존성 관리 |
| **배포** | Docker | 컨테이너화 |

## 3. 디렉토리 구조

```
MiroFish-Ko/
├── backend/
│   ├── run.py                          # Flask 진입점
│   ├── pyproject.toml                  # Python 의존성 (uv)
│   └── app/
│       ├── __init__.py                 # Flask 앱 팩토리
│       ├── config.py                   # 전역 설정 (.env 기반)
│       ├── api/                        # REST API 라우트
│       │   ├── graph.py                #   온톨로지 + 그래프 API
│       │   ├── simulation.py           #   시뮬레이션 API
│       │   └── report.py              #   보고서 + 대화 API
│       ├── models/
│       │   ├── project.py              #   프로젝트 메타데이터 관리
│       │   └── task.py                 #   비동기 작업 추적
│       ├── services/                   # 핵심 비즈니스 로직
│       │   ├── ontology_generator.py   #   LLM → 온톨로지
│       │   ├── graph_builder.py        #   Zep 그래프 구축
│       │   ├── zep_entity_reader.py    #   Zep 엔터티 추출
│       │   ├── oasis_profile_generator.py  # 에이전트 프로필 생성
│       │   ├── simulation_manager.py   #   시뮬레이션 준비 조율
│       │   ├── simulation_config_generator.py  # 설정 자동 생성
│       │   ├── simulation_runner.py    #   OASIS 실행 관리
│       │   ├── report_agent.py         #   ReportAgent (도구 활용)
│       │   ├── zep_tools.py            #   ReportAgent 도구 세트
│       │   ├── zep_graph_memory_updater.py  # 메모리 업데이트
│       │   ├── text_processor.py       #   텍스트 청크 분할
│       │   └── simulation_ipc.py       #   프로세스 간 통신
│       └── utils/
│           ├── llm_client.py           #   OpenAI SDK 호환 클라이언트
│           ├── file_parser.py          #   PDF/MD/TXT 파싱
│           ├── logger.py               #   구조화된 로깅
│           └── retry.py                #   재시도 데코레이터
├── frontend/
│   ├── src/
│   │   ├── main.js                     # Vue 진입점
│   │   ├── App.vue                     # 루트 컴포넌트
│   │   ├── api/                        # Axios API 클라이언트
│   │   ├── views/                      # 페이지별 뷰
│   │   │   ├── Home.vue                #   홈 (프로젝트 생성)
│   │   │   ├── MainView.vue            #   메인 워크플로우
│   │   │   ├── Process.vue             #   프로세스 진행 상황
│   │   │   ├── SimulationView.vue      #   시뮬레이션 화면
│   │   │   ├── SimulationRunView.vue   #   시뮬레이션 실행
│   │   │   ├── ReportView.vue          #   보고서 열람
│   │   │   └── InteractionView.vue     #   에이전트 대화
│   │   └── components/                 # 재사용 컴포넌트
│   │       ├── GraphPanel.vue          #   D3 그래프 시각화
│   │       ├── Step2EnvSetup.vue       #   환경 설정
│   │       ├── Step3Simulation.vue     #   시뮬레이션 설정
│   │       ├── Step4Report.vue         #   보고서 생성 타임라인
│   │       └── Step5Interaction.vue    #   심화 상호작용
│   └── vite.config.js                  # Vite 설정
├── docker-compose.yml                  # Docker 컨테이너 설정
├── Dockerfile                          # 이미지 빌드
├── .env.example                        # 환경변수 템플릿
└── package.json                        # 루트 npm scripts
```

## 4. API 엔드포인트

### 그래프/온톨로지 (`/api/graph`)

| 메서드 | 엔드포인트 | 설명 |
|--------|-----------|------|
| POST | `/ontology/generate` | 파일 업로드 → 온톨로지 생성 |
| POST | `/build` | 온톨로지 → Zep 그래프 구축 (비동기) |
| GET | `/task/<task_id>` | 작업 상태 조회 |
| GET | `/data/<graph_id>` | 그래프 노드/엣지 조회 |
| DELETE | `/delete/<graph_id>` | 그래프 삭제 |
| GET | `/project/list` | 프로젝트 목록 |
| DELETE | `/project/<id>` | 프로젝트 삭제 |

### 시뮬레이션 (`/api/simulation`)

| 메서드 | 엔드포인트 | 설명 |
|--------|-----------|------|
| GET | `/entities/<graph_id>` | 그래프 엔터티 조회 |
| POST | `/prepare` | 시뮬레이션 환경 준비 |
| GET | `/status/<sim_id>` | 상태 조회 |
| POST | `/run` | OASIS 시뮬레이션 실행 |

### 보고서 (`/api/report`)

| 메서드 | 엔드포인트 | 설명 |
|--------|-----------|------|
| POST | `/generate` | 보고서 생성 시작 |
| POST | `/generate/status` | 생성 진행률 |
| GET | `/<report_id>` | 보고서 조회 |
| POST | `/chat` | ReportAgent 대화 |
| GET | `/<id>/agent-log` | Agent 로그 |
| GET | `/<id>/sections` | 섹션 목록 |

## 5. 핵심 서비스 모듈

| 모듈 | 크기 | 역할 |
|------|------|------|
| **report_agent.py** | 87KB | ReportAgent: 도구 활용 보고서 생성 + 대화 |
| **simulation_runner.py** | 64KB | OASIS 시뮬레이션 프로세스 관리 |
| **oasis_profile_generator.py** | 44KB | Zep 엔터티 → Agent Profile (병렬 생성) |
| **simulation_config_generator.py** | 36KB | LLM 기반 시뮬레이션 설정 자동 생성 |
| **simulation_manager.py** | 21KB | 시뮬레이션 준비 3단계 조율 |
| **zep_graph_memory_updater.py** | 19KB | 시뮬레이션 중 Zep 메모리 업데이트 |
| **graph_builder.py** | 18KB | Zep Cloud 그래프 구축/관리 |
| **zep_entity_reader.py** | 15KB | Zep 그래프에서 엔터티 추출/필터링 |
| **ontology_generator.py** | 14KB | LLM → 온톨로지 (엔터티/관계 타입) |

## 6. 외부 서비스 연동

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  LLM API    │     │  Zep Cloud   │     │   OASIS     │
│ (Gemini 등) │     │ (메모리그래프)│     │ (시뮬레이션) │
└──────┬──────┘     └──────┬───────┘     └──────┬──────┘
       │                   │                    │
       │ OpenAI SDK        │ REST API           │ Python
       │ 호환 형식         │ (Python SDK)       │ subprocess
       │                   │                    │
       ▼                   ▼                    ▼
┌──────────────────────────────────────────────────────┐
│                  Flask Backend (5001)                  │
│                                                       │
│  ontology_generator  graph_builder  simulation_runner │
│  profile_generator   report_agent   memory_updater    │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│                Vue 3 Frontend (3000)                   │
│                                                       │
│  Home → GraphPanel → SimulationView → ReportView     │
└──────────────────────────────────────────────────────┘
```

## 7. 데이터 저장 구조

```
backend/uploads/
├── projects/proj_xxxx/
│   ├── project.json           # 메타데이터
│   ├── extracted_text.txt     # 추출 텍스트
│   └── files/                 # 원본 파일
├── simulations/sim_xxxx/
│   ├── state.json             # 상태 (CREATED→PREPARING→READY)
│   ├── twitter_profiles.csv   # Twitter 에이전트 프로필
│   ├── reddit_profiles.json   # Reddit 에이전트 프로필
│   └── simulation_config.json # 시뮬레이션 설정
└── reports/report_xxxx/
    ├── report.json            # 메타데이터
    ├── report.md              # 전체 마크다운
    ├── section_01.md ~ N.md   # 섹션별 파일
    ├── outline.json           # 보고서 구조
    ├── agent_log.jsonl        # Agent 로그 (JSON Lines)
    └── console_log.txt        # 콘솔 출력
```

## 8. 주요 설정값

```python
# LLM
LLM_API_KEY          # API 키
LLM_BASE_URL         # 엔드포인트 URL
LLM_MODEL_NAME       # 모델명

# Zep
ZEP_API_KEY          # Zep Cloud API 키

# 온톨로지
ENTITY_TYPES = 10    # 엔터티 타입 수 (고정)
EDGE_TYPES = 6~10    # 관계 타입 수

# 텍스트 처리
CHUNK_SIZE = 500     # 청크 크기
CHUNK_OVERLAP = 50   # 청크 오버랩

# 시뮬레이션
MAX_ROUNDS = 10      # 기본 라운드 수

# ReportAgent
MAX_TOOL_CALLS = 5           # 도구 호출 최대 횟수
MAX_REFLECTION_ROUNDS = 2    # 리플렉션 라운드
TEMPERATURE = 0.5            # LLM 온도

# 파일
MAX_UPLOAD = 50MB
ALLOWED = pdf, md, txt
```

## 9. 디자인 패턴 요약

| 패턴 | 적용 위치 | 설명 |
|------|----------|------|
| **비동기 작업** | TaskManager | 그래프 구축, 프로필 생성, 보고서 등 장시간 작업 |
| **병렬 처리** | ThreadPoolExecutor | 프로필 생성 시 3-5개 동시 처리 |
| **LLM 재시도** | retry 데코레이터 | 온톨로지/프로필/보고서 생성 시 최대 3회 |
| **도구 활용** | ReportAgent | LLM이 도구(검색/통계)를 선택 실행 |
| **상태 머신** | SimulationManager | CREATED→PREPARING→READY/FAILED |
| **파일 기반 영속성** | uploads/ | JSON/CSV/MD 파일로 상태 저장 |
| **청크 처리** | TextProcessor | 대용량 텍스트를 LLM 입력 크기로 분할 |

## 10. 다른 프로젝트 참고 포인트

### LLM 연동 패턴
- OpenAI SDK 호환 형식으로 통일 → 모델 교체 용이
- `llm_client.py`에서 단일 클라이언트 관리
- 시스템 프롬프트로 출력 형식 강제 (JSON 스키마)

### 멀티 에이전트 패턴
- 에이전트 = 프로필(성격/기억) + 행동 로직
- Zep Cloud로 장기 기억 관리 (GraphRAG)
- 라운드 기반 시뮬레이션 (턴제)

### 보고서 생성 패턴 (ReportAgent)
- LLM + 도구 활용 (Tool Use)
- 섹션별 생성 → 전체 조합
- 리플렉션 라운드로 품질 향상
- 실시간 로그 스트리밍 (agent_log.jsonl)

### Docker 구성
- 단일 컨테이너에 프론트/백 동시 실행 (concurrently)
- 소스 빌드 방식 (Dockerfile에서 Node + Python 설치)
- volumes로 업로드 디렉토리 영속화

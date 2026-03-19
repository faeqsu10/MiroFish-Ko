<div align="center">

<img src="./static/image/MiroFish_logo_compressed.jpeg" alt="MiroFish Logo" width="75%"/>

</br>
<em>간단하고 범용적인 군집 지능 엔진, 무엇이든 예측합니다</em>

[![GitHub Stars](https://img.shields.io/github/stars/faeqsu10/MiroFish-Ko?style=flat-square&color=DAA520)](https://github.com/faeqsu10/MiroFish-Ko/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/faeqsu10/MiroFish-Ko?style=flat-square)](https://github.com/faeqsu10/MiroFish-Ko/network)
[![Docker](https://img.shields.io/badge/Docker-Build-2496ED?style=flat-square&logo=docker&logoColor=white)](https://hub.docker.com/)

[한국어](./README.md) | [English](./README-EN.md) | [中文](./README-ZH.md)

</div>

## 개요

**MiroFish**는 멀티 에이전트 기술로 구동되는 차세대 AI 예측 엔진입니다. 속보, 정책 초안, 금융 신호 같은 현실 세계의 시드 정보를 추출해 고충실도의 병렬 디지털 세계를 자동으로 구성합니다. 이 공간에서는 독립적인 성격, 장기 기억, 행동 로직을 갖춘 수천 개의 지능형 에이전트가 자유롭게 상호작용하며 사회적으로 진화합니다. 사용자는 "신의 시점"에서 변수를 동적으로 주입해 미래의 전개를 정밀하게 추론할 수 있습니다.

> 사용자가 할 일은 단 두 가지입니다: 시드 자료(데이터 분석 보고서 또는 흥미로운 소설 스토리)를 업로드하고, 자연어로 예측 요구사항을 설명하세요.</br>
> MiroFish가 반환하는 결과: 상세한 예측 리포트와 깊이 있게 상호작용 가능한 고충실도 디지털 세계

### 비전

MiroFish는 현실을 비추는 군집 지능 미러를 만드는 것을 목표로 합니다.

- **거시적 관점**: 의사결정자를 위한 리허설 실험실 — 정책과 PR 전략을 무위험으로 사전 검증
- **미시적 관점**: 개인 사용자를 위한 창작 샌드박스 — 소설 결말 추론부터 상상 시나리오 탐색까지

## 한국어 포크 변경사항

이 저장소는 [666ghj/MiroFish](https://github.com/666ghj/MiroFish)의 한국어 포크([ByeongkiJeong/MiroFish-Ko](https://github.com/ByeongkiJeong/MiroFish-Ko))를 기반으로 추가 한국어화를 적용한 버전입니다.

- Docker 이미지를 소스 빌드로 전환 (중국어 원본 이미지 제거)
- 폰트: `Noto Sans SC` → `Noto Sans KR` (한국어 최적화)
- 날짜 로캘: `zh-CN` → `ko-KR`
- 시간대 설정: `CHINA_TIMEZONE_CONFIG` → `KST_TIMEZONE_CONFIG`
- Dockerfile: NodeSource 20 LTS 설치 방식으로 변경

## 워크플로우

1. **그래프 구축**: 시드 추출 & 개인/집단 메모리 주입 & GraphRAG 구축
2. **환경 설정**: 엔터티 관계 추출 & 페르소나 생성 & 에이전트 설정 주입
3. **시뮬레이션**: 듀얼 플랫폼 병렬 시뮬레이션 & 예측 요구사항 자동 파싱 & 동적 시계열 메모리 업데이트
4. **리포트 생성**: 풍부한 도구 세트를 갖춘 ReportAgent로 시뮬레이션 후 환경과 심층 상호작용
5. **심화 상호작용**: 시뮬레이션 세계의 모든 에이전트와 대화 & ReportAgent와 상호작용

## 빠른 시작

### 사전 요구사항

| 도구 | 버전 | 설명 | 설치 확인 |
|------|------|------|-----------|
| **Docker** | 최신 | 컨테이너 런타임 | `docker --version` |
| **Git** | 최신 | 소스 클론 | `git --version` |

### 필요한 API 키

| 서비스 | 용도 | 발급처 |
|--------|------|--------|
| **LLM API** | AI 에이전트 구동 | [Google AI Studio](https://aistudio.google.com/apikey) (Gemini 권장) |
| **Zep API** | 에이전트 메모리 그래프 | [Zep Cloud](https://app.getzep.com/) (무료 플랜 가능) |

### 설치 및 실행

```bash
# 1. 저장소 클론
git clone https://github.com/faeqsu10/MiroFish-Ko.git
cd MiroFish-Ko

# 2. 환경 변수 설정
cp .env.example .env
```

`.env` 파일을 열어 아래 값을 입력합니다:

```env
# LLM API 설정 (OpenAI SDK 호환 형식)
# Gemini 사용 예시:
LLM_API_KEY=your_gemini_api_key
LLM_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
LLM_MODEL_NAME=gemini-2.5-flash

# Zep Cloud 설정
ZEP_API_KEY=your_zep_api_key
```

```bash
# 3. Docker로 빌드 및 실행 (소스에서 빌드)
docker compose up -d
```

빌드 완료 후 브라우저에서 접속:
- **프런트엔드**: http://localhost:3000
- **백엔드 API**: http://localhost:5001

### 소스 코드 직접 실행 (Docker 없이)

```bash
# 사전 요구: Node.js 18+, Python 3.11/3.12, uv

# 전체 의존성 설치
npm run setup:all

# 서비스 시작
npm run dev
```

## 지원 LLM 모델

OpenAI SDK 호환 형식을 따르는 모든 LLM API를 사용할 수 있습니다:

| 모델 | `LLM_BASE_URL` | `LLM_MODEL_NAME` |
|------|-----------------|-------------------|
| **Gemini** (권장) | `https://generativelanguage.googleapis.com/v1beta/openai` | `gemini-2.5-flash` |
| **OpenAI** | `https://api.openai.com/v1` | `gpt-4o` |
| **Claude** | `https://api.anthropic.com/v1` | `claude-sonnet-4-20250514` |
| **Alibaba Qwen** | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-plus` |

> 비용이 클 수 있으므로 처음에는 **40라운드 미만**으로 시뮬레이션을 권장합니다.

## 감사의 글

- 원본 프로젝트: [MiroFish](https://github.com/666ghj/MiroFish) by 666ghj (Shanda Group 지원)
- 한국어 번역: [MiroFish-Ko](https://github.com/ByeongkiJeong/MiroFish-Ko) by ByeongkiJeong
- 시뮬레이션 엔진: [OASIS](https://github.com/camel-ai/oasis) by CAMEL-AI

# CLAUDE.md

이 파일은 Claude Code (claude.ai/code)가 이 저장소에서 작업할 때 참고하는 가이드 문서입니다.

## 기본 설정

- **모든 대화는 한국어로 진행합니다**
- 문서 작성 및 수정 시에도 한국어를 사용합니다

---

## 저장소 개요

이 저장소는 **내시피(Naecipe)** 서비스의 기획 및 아키텍처 문서 저장소입니다.

**내시피(Naecipe)**: "나만의 레시피"를 의미하는 AI 기반 개인화 요리 플랫폼. 사용자가 외부 레시피를 따라 요리하고, 피드백을 입력하면 AI가 개인 취향에 맞게 레시피를 보정해주는 것이 핵심 가치입니다.

---

## 문서 구조

```
PLANS/
├── PLAN_GUIDE.md              # 문서 작성 가이드
├── CLAUDE.md                  # 본 문서 (Claude Code 가이드)
├── FEEDBACK_REPORT.md         # 문서 종합 피드백 보고서
│
├── [1단계: 서비스/사업 방향]
│   ├── 1-1SERVICE_PLAN.md     # 서비스 기획서
│   └── 1-2BUSINESS_PLAN.md    # 사업계획서
│
├── [2단계: 요구사항 및 명세]
│   ├── PRD.md                 # 제품 요구사항 문서
│   ├── 2-1REQUIREMENT.md      # 요구사항 정의서
│   └── 2-2FEATURE_SPEC.md     # 기능 명세서
│
├── [3단계: UX/UI 설계]
│   ├── 3-1IA_MENU_STRUCTURE.md   # IA/메뉴 구조도
│   ├── 3-2USER_SCENARIO.md       # 사용자 시나리오
│   └── 3-3WIREFRAME.md           # 화면 설계서/와이어프레임
│
├── [4단계: 정책]
│   └── 4.1TERMS_OF_SERVICE.md    # 서비스 정책서
│
└── [5단계: 아키텍처]
    ├── 5-1SERVICE_ARCHITECTURE.md   # 아키텍처 INDEX
    ├── 5-1-1_DOMAIN.md              # 도메인 분석
    ├── 5-1-2_SYSTEM.md              # 시스템 아키텍처
    ├── 5-1-3_AI_AGENT.md            # AI 에이전트
    ├── 5-1-4_API.md                 # API 설계
    ├── 5-1-5_FRONTEND.md            # 프론트엔드
    ├── 5-1-6_INFRA.md               # 인프라 및 배포
    ├── 5-1-7_SECURITY.md            # 보안 및 품질
    ├── 5-1-8_OPERATIONS.md          # 운영 가이드
    └── 5-1-9_ADR.md                 # 아키텍처 결정 기록
```

---

## Core Loop (핵심 비즈니스 플로우)

```
[데이터 파이프라인]
외부 소스(YouTube/Instagram/Blog) → Crawler Bot → LLM 파싱 → 중복 검사 → Ingestion API → Recipe DB

[사용자 Core Loop]
검색 → 레시피 상세 → 조리 시작 → 피드백 입력 → AI 보정 → 레시피북 저장 → (다음 조리 시 반복)
```

---

## 기술 스택

| 영역 | 기술 | 비고 |
|------|------|------|
| **백엔드** | Python (FastAPI) | 고성능, 타입 힌트, AI/ML 통합 용이 |
| **프론트엔드** | Next.js 14 (App Router) | SSR/SSG, React 생태계 |
| **데이터베이스** | PostgreSQL | 도메인별 5개 DB 분리, pgvector |
| **캐시** | Redis Cluster | 세션 관리, 다층 캐시 |
| **메시징** | Apache Kafka (AWS MSK) | 이벤트 소싱, 비동기 처리 |
| **AI 에이전트** | LangGraph + OpenAI/Anthropic | 보정 Agent, Q&A Agent, Crawler Agent |
| **크롤러** | LangGraph + Playwright | 레시피 수집, LLM 파싱 |
| **인프라** | AWS EKS | Kubernetes 오케스트레이션 |
| **CI/CD** | GitHub Actions + ArgoCD | GitOps 기반 배포 |
| **모니터링** | Prometheus + Grafana + Loki | 메트릭, 로그, 트레이싱 |

---

## 도메인별 데이터베이스

| DB | 용도 | 주요 테이블 |
|----|------|------------|
| **Recipe DB** | 레시피 데이터 | recipes, ingredients, cooking_steps, tags, recipe_sources |
| **User DB** | 사용자/인증 | users, user_profiles, oauth_accounts, taste_preferences |
| **Cookbook DB** | 레시피북/피드백 | cookbooks, cookbook_recipes, recipe_versions, cooking_feedbacks |
| **Knowledge DB** | AI RAG 지식 | knowledge_chunks, embeddings (pgvector) |
| **Analytics DB** | 분석 데이터 | events, aggregations (TimescaleDB) |

---

## 서비스 구성

### 백엔드 서비스

| 서비스 | 포트 | 역할 |
|--------|------|------|
| API Gateway | 8000 | 라우팅, 인증, Rate Limiting |
| Recipe Service | 8001 | 레시피 CRUD, 검색 |
| User Service | 8002 | 인증/인가, 프로필 |
| Cookbook Service | 8003 | 레시피북, 피드백 관리 |
| AI Agent Service | 8004 | 보정 Agent, Q&A Agent |
| Ingestion Service | 8009 | 크롤러 데이터 수신, 중복 검사 |
| Analytics Service | 8010 | 이벤트 수집, 집계 |

### AI Agents (LangGraph)

| Agent | 역할 |
|-------|------|
| **Adjustment Agent** | 피드백 분석 → RAG 검색 → 레시피 보정 생성 |
| **Q&A Agent** | 요리 중 질문 응답 (재료 대체, 보관법 등) |
| **Recipe Crawler Agent** | 외부 플랫폼 레시피 수집 → LLM 파싱 → 정규화 |

---

## 주요 용어 (Glossary)

| 용어 | 설명 |
|------|------|
| **원본 레시피** | 외부 출처에서 크롤링하여 수집한 기본 레시피 |
| **보정 레시피** | 사용자 피드백 기반으로 AI가 튜닝한 개인화 레시피 |
| **레시피북 (Cookbook)** | 사용자가 저장한 레시피 모음 (원본 + 보정 버전) |
| **Cookbook Recipe** | 레시피북에 저장된 개별 레시피 (버전 히스토리 포함) |
| **신뢰도/인기도 스코어** | 플랫폼 지표(조회수, 좋아요) + 내부 지표 조합 점수 |
| **취향 프로파일** | 사용자의 맛 선호도 (단맛, 짠맛, 매운맛, 신맛 1-5) |
| **Core Loop** | 검색 → 상세 → 조리 → 피드백 → AI 보정 → 저장의 핵심 사용 흐름 |
| **Ingestion** | 크롤러가 수집한 데이터를 검증 후 DB에 저장하는 과정 |

---

## 화면/기능 ID 체계

| ID | 화면/기능 |
|----|----------|
| `G-01/G-02` | 공통 컴포넌트 (헤더, 토스트) |
| `H-01~H-03` | 홈 & 검색 화면 |
| `S-01~S-03` | 검색 결과 리스트 |
| `R-01~R-05` | 레시피 상세 화면 |
| `C-01/C-02` | 조리 플로우 모달 |
| `A-01/A-02` | AI 보정 화면 |
| `B-01~B-03` | 레시피북 화면 |
| `Q-01/Q-02` | Q&A 에이전트 화면 |
| `A-U-01/A-U-02` | 계정/세션 화면 |

---

## User Story ID 체계

| Epic | ID Prefix | 예시 |
|------|-----------|------|
| 통합 검색 & 탐색 | US-SEARCH-XX | US-SEARCH-01 |
| 레시피 상세 & 조리 | US-DETAIL-XX | US-DETAIL-01 |
| 레시피북 저장 & 관리 | US-BOOK-XX | US-BOOK-01 |
| 피드백 & AI 보정 | US-ADJ-XX | US-ADJ-01 |
| 요리 Q&A 에이전트 | US-QNA-XX | US-QNA-01 |
| 계정 & 세션 | US-AUTH-XX | US-AUTH-01 |

---

## API 엔드포인트 개요

### Public API (REST)

```
/api/v1/recipes          # 레시피 검색, 상세
/api/v1/auth             # 인증 (로그인, 회원가입, OAuth)
/api/v1/users            # 사용자 프로필, 취향 설정
/api/v1/cookbooks        # 레시피북 CRUD
/api/v1/ai/adjustments   # AI 보정 상태 조회
/api/v1/ai/qa            # Q&A 질문/응답
```

### Internal API

```
/api/v1/ingestion        # 크롤러 → 레시피 등록 (Internal)
gRPC                     # 서비스 간 통신 (GetUser, GetRecipe 등)
```

---

## 이벤트 토픽 (Kafka)

| 토픽 | 발행자 | 소비자 |
|------|--------|--------|
| `recipe.events` | Recipe Service | Analytics, Search |
| `user.events` | User Service | Analytics, Notification |
| `cookbook.events` | Cookbook Service | Analytics |
| `feedback.events` | Cookbook Service | AI Agent |
| `ai.events` | AI Agent | Notification, Analytics |

---

## Phase 1 MVP 범위

### Must (필수)
- 통합 레시피 검색 (키워드, 필터)
- 레시피 상세 보기
- 레시피북 저장/조회
- 조리 기록 (시작/완료)
- 피드백 입력 및 AI 보정
- Q&A 요리 에이전트
- 이메일/소셜 로그인

### Should (권장)
- 최근 검색어/최근 본 레시피
- 레시피북 폴더/라벨 분류
- 원본/보정 비교 뷰
- Q&A 답변 평가

### Phase 2+ (향후)
- 유저 레시피 게시
- 크리에이터 대시보드
- 프리미엄 요금제

---

## 품질 속성 목표

| 속성 | 목표 |
|------|------|
| 가용성 | 99.9% (월 43분 이하 다운타임) |
| 검색 응답 시간 | < 200ms (p99) |
| 상세 응답 시간 | < 100ms (p99) |
| AI 보정 시간 | < 10초 |
| 동시 사용자 | Phase 1: 50,000명 |
| RPO | < 5분 |
| RTO | < 30분 |

---

## 문서 작업 시 유의사항

1. **번호 체계 유지**: `PLAN_GUIDE.md`의 문서 번호 체계 준수
2. **용어 일관성**: 위 Glossary의 용어 사용
3. **기술 스택 일관성**: FastAPI, Next.js 14, PostgreSQL 등 확정된 스택 사용
4. **Phase 분류**: Must/Should/Not-in-P1 분류 체계 준수
5. **ID 체계**: 화면 ID(G-01), User Story ID(US-SEARCH-01) 형식 준수
6. **참조 링크**: 관련 문서 참조 시 파일명 명시

---

## 팀별 필수 문서

| 팀 | 필수 문서 |
|----|----------|
| **백엔드 개발** | 5-1-2_SYSTEM, 5-1-3_AI_AGENT, 5-1-4_API |
| **프론트엔드 개발** | 5-1-5_FRONTEND, 5-1-4_API, 3-3WIREFRAME |
| **AI/ML** | 5-1-3_AI_AGENT |
| **인프라/DevOps** | 5-1-6_INFRA, 5-1-8_OPERATIONS |
| **보안** | 5-1-7_SECURITY |
| **QA** | 5-1-7_SECURITY, 5-1-4_API, 3-2USER_SCENARIO |
| **PM/기획** | 1-1SERVICE_PLAN, PRD, 3-2USER_SCENARIO |

---

## 주요 다이어그램 위치

| 다이어그램 | 문서 |
|-----------|------|
| Core Data Flow | 5-1-1_DOMAIN |
| 시스템 아키텍처 | 5-1-2_SYSTEM |
| ER 다이어그램 | 5-1-7_SECURITY |
| AI Agent Flow | 5-1-3_AI_AGENT |
| Crawler Agent Flow | 5-1-3_AI_AGENT |
| AWS 인프라 | 5-1-6_INFRA |
| CI/CD 파이프라인 | 5-1-6_INFRA |
| DR 아키텍처 | 5-1-8_OPERATIONS |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|-----|------|----------|
| v1.0 | 2025.11.25 | 초기 작성 (기획 문서 기준) |
| v2.0 | 2025.11.30 | 아키텍처 문서 통합, 기술 스택/API/이벤트 추가 |

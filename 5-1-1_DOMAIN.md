# 내시피(Naecipe) 도메인 분석

> 상위 문서: [5-1SERVICE_ARCHITECTURE.md](./5-1SERVICE_ARCHITECTURE.md)

---

## 1. 핵심 데이터 흐름 (Core Data Flow)

내시피 서비스의 핵심은 **Core Loop**이다. 사용자가 레시피를 검색하고, 조리하고, 피드백을 남기면 AI가 개인화된 레시피를 제공하는 선순환 구조를 형성한다.

### 1.1 Core Loop 다이어그램

```mermaid
flowchart LR
    subgraph User["👤 사용자"]
        A[검색]
        B[레시피 상세 보기]
        C[조리 시작]
        D[피드백 입력]
    end

    subgraph AI["🤖 AI 시스템"]
        E[AI 보정]
    end

    subgraph Storage["💾 저장소"]
        F[레시피북 저장]
    end

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F -.->|다음 조리 시| B

    style A fill:#e1f5fe
    style E fill:#fff3e0
    style F fill:#e8f5e9
```

### 1.2 원본 레시피 수집 플로우

Core Loop가 작동하기 위해서는 사용자가 검색할 수 있는 **원본 레시피 데이터베이스**가 구축되어 있어야 한다. 이 데이터는 외부 플랫폼에서 크롤링하여 수집된다.

```mermaid
flowchart TB
    subgraph External["🌐 외부 레시피 소스"]
        YT[YouTube<br/>유명 쉐프 채널]
        IG[Instagram<br/>요리 인플루언서]
        BLOG[블로그<br/>네이버, 티스토리]
        SITE[레시피 사이트<br/>만개의레시피 등]
    end

    subgraph Crawler["🤖 Recipe Crawler Bot"]
        C1[소스 탐색]
        C2[콘텐츠 추출]
        C3[LLM 파싱]
        C4[중복 검사]
    end

    subgraph Ingestion["📥 Ingestion Service"]
        I1[스키마 검증]
        I2[유사도 검사]
        I3[스코어 계산]
        I4[DB 저장/갱신]
    end

    subgraph RecipeDB["💾 Recipe DB"]
        DB[(원본 레시피)]
    end

    External --> C1
    C1 --> C2
    C2 --> C3
    C3 --> C4
    C4 --> I1
    I1 --> I2
    I2 --> I3
    I3 --> I4
    I4 --> DB

    DB -.->|사용자 검색| CoreLoop[Core Loop]

    style CoreLoop fill:#e8f5e9
```

### 1.3 상세 데이터 흐름

```mermaid
flowchart TB
    subgraph Search["1️⃣ 검색 단계"]
        S1[키워드 검색] --> S2[필터 적용]
        S2 --> S3[검색 결과]
    end

    subgraph Detail["2️⃣ 상세 조회 단계"]
        D1[레시피 상세] --> D2[원본 vs 보정본 비교]
        D2 --> D3[조리 시작 결정]
    end

    subgraph Cook["3️⃣ 조리 단계"]
        C1[타이머 시작] --> C2[단계별 진행]
        C2 --> C3[조리 완료]
    end

    subgraph Feedback["4️⃣ 피드백 단계"]
        F1[맛 평가] --> F2[텍스트 피드백]
        F2 --> F3[개선 요청]
    end

    subgraph AI["5️⃣ AI 보정 단계"]
        A1[피드백 분석] --> A2[레시피 보정]
        A2 --> A3[보정본 생성]
    end

    subgraph Save["6️⃣ 저장 단계"]
        V1[레시피북 저장] --> V2[버전 관리]
        V2 --> V3[히스토리 기록]
    end

    Search --> Detail
    Detail --> Cook
    Cook --> Feedback
    Feedback --> AI
    AI --> Save
    Save -.->|반복| Detail
```

---

## 2. 도메인 경계 식별

### 2.1 Bounded Context Map

```mermaid
flowchart TB
    subgraph RecipeDomain["🍳 Recipe Domain"]
        R1[Recipe Service]
        R2[Search Service]
        R3[Recipe DB]
    end

    subgraph UserDomain["👤 User Domain"]
        U1[User Service]
        U2[Profile Service]
        U3[User DB]
    end

    subgraph CookbookDomain["📚 Cookbook Domain"]
        CB1[Cookbook Service]
        CB2[Version Service]
        CB3[Cookbook DB]
    end

    subgraph AIDomain["🤖 AI Agent Domain"]
        AI1[Adjustment Agent]
        AI2[Q&A Agent]
        AI3[Tagging Agent]
        AI4[Knowledge DB]
    end

    subgraph AnalyticsDomain["📊 Analytics Domain"]
        AN1[Event Collector]
        AN2[Analytics Service]
        AN3[Analytics DB]
    end

    RecipeDomain <-->|레시피 정보| CookbookDomain
    UserDomain <-->|사용자 정보| CookbookDomain
    CookbookDomain -->|피드백 데이터| AIDomain
    AIDomain -->|보정된 레시피| CookbookDomain

    RecipeDomain -->|이벤트| AnalyticsDomain
    UserDomain -->|이벤트| AnalyticsDomain
    CookbookDomain -->|이벤트| AnalyticsDomain
    AIDomain -->|이벤트| AnalyticsDomain
```

### 2.2 도메인별 책임

| 도메인 | 핵심 책임 | 주요 엔티티 |
|--------|----------|------------|
| **Recipe** | 외부 레시피 수집, 검색, 정규화, 스코어링 | Recipe, RecipeSource, Ingredient, Step, Tag |
| **Recipe Ingestion** | 크롤링 데이터 수신, 중복 검사, 점수 갱신 | RecipeSource, ScoreHistory |
| **User** | 회원 관리, 인증, 프로필 | User, Profile, Preference |
| **Cookbook** | 개인 레시피북, 버전 관리 | Cookbook, CookbookRecipe, Version, Feedback |
| **AI Agent** | 레시피 보정, Q&A, 자동 태깅, 크롤링 | AdjustmentRequest, KnowledgeBase, Embedding |
| **Analytics** | 사용자 행동 분석, 통계 | Event, Session, Metric |

---

## 3. 도메인 상세 정의

### 3.1 Recipe Domain

**목적:** 외부 레시피를 수집하고 정규화하여 검색 가능한 형태로 제공

```mermaid
erDiagram
    RECIPE {
        uuid id PK
        string title
        string author_name
        string author_channel
        string source_url
        string source_platform
        text description
        int cooking_time_minutes
        int servings
        string difficulty
        decimal quality_score
        decimal popularity_score
        decimal exposure_score
        string content_hash
        boolean is_verified
        timestamp created_at
        timestamp updated_at
    }

    RECIPE_SOURCE {
        uuid id PK
        uuid recipe_id FK
        string platform
        string source_url UK
        string original_title
        bigint platform_view_count
        int platform_like_count
        int platform_comment_count
        jsonb raw_data
        timestamp first_discovered_at
        timestamp last_updated_at
    }

    INGREDIENT {
        uuid id PK
        uuid recipe_id FK
        string name
        string amount
        string unit
        int order_index
        boolean is_optional
    }

    COOKING_STEP {
        uuid id PK
        uuid recipe_id FK
        int step_number
        text instruction
        int duration_seconds
        string step_type
    }

    TAG {
        uuid id PK
        string name
        string category
    }

    RECIPE_TAG {
        uuid recipe_id FK
        uuid tag_id FK
    }

    RECIPE ||--o{ RECIPE_SOURCE : crawled_from
    RECIPE ||--o{ INGREDIENT : contains
    RECIPE ||--o{ COOKING_STEP : has
    RECIPE ||--o{ RECIPE_TAG : tagged_with
    TAG ||--o{ RECIPE_TAG : applied_to
```

**핵심 기능:**
- **외부 레시피 수집**: Crawler Bot이 유명 쉐프/인플루언서 레시피를 크롤링
- **중복 검사**: 제목+저자 해시 및 콘텐츠 유사도(임베딩)로 중복 판단
- **스코어링**: 인기도, 품질, 신선도 기반 노출 점수 산정
- **전문 검색**: Elasticsearch 연동
- **태그 기반 필터링**
- **인기도/평점 기반 정렬**

**데이터 수집 흐름:**
1. Crawler Bot이 YouTube, Instagram, 블로그 등에서 레시피 콘텐츠 발견
2. LLM이 비정형 콘텐츠를 구조화된 레시피 형태로 파싱
3. Ingestion Service가 중복 검사 수행 (제목+저자 / 콘텐츠 유사도)
4. 신규 레시피면 저장, 기존이면 노출 점수 갱신

### 3.2 User Domain

**목적:** 사용자 계정 관리, 인증/인가, 개인 설정

```mermaid
erDiagram
    USER {
        uuid id PK
        string email UK
        string password_hash
        string name
        string profile_image_url
        timestamp created_at
        timestamp last_login_at
        boolean is_active
    }

    USER_PROFILE {
        uuid id PK
        uuid user_id FK
        jsonb dietary_restrictions
        jsonb allergies
        jsonb cuisine_preferences
        int skill_level
        int household_size
    }

    TASTE_PREFERENCE {
        uuid id PK
        uuid user_id FK
        string category
        int sweetness
        int saltiness
        int spiciness
        int sourness
        int umami
    }

    USER ||--|| USER_PROFILE : has
    USER ||--o{ TASTE_PREFERENCE : prefers
```

**핵심 기능:**
- OAuth 2.0 소셜 로그인 (Google, Kakao, Naver)
- 세션 관리 (JWT + Redis)
- 취향 프로필 관리
- 알레르기/식이 제한 설정

### 3.3 Cookbook Domain

**목적:** 개인 레시피북 관리, 버전 이력, 피드백 수집

```mermaid
erDiagram
    COOKBOOK {
        uuid id PK
        uuid user_id FK
        string name
        string description
        boolean is_default
        timestamp created_at
    }

    COOKBOOK_RECIPE {
        uuid id PK
        uuid cookbook_id FK
        uuid original_recipe_id FK
        jsonb adjusted_data
        int cook_count
        float personal_rating
        timestamp last_cooked_at
        timestamp created_at
    }

    RECIPE_VERSION {
        uuid id PK
        uuid cookbook_recipe_id FK
        int version_number
        jsonb recipe_snapshot
        string change_summary
        timestamp created_at
    }

    COOKING_FEEDBACK {
        uuid id PK
        uuid cookbook_recipe_id FK
        uuid version_id FK
        int taste_rating
        int difficulty_rating
        text feedback_text
        jsonb adjustment_requests
        timestamp created_at
    }

    COOKBOOK ||--o{ COOKBOOK_RECIPE : contains
    COOKBOOK_RECIPE ||--o{ RECIPE_VERSION : versions
    COOKBOOK_RECIPE ||--o{ COOKING_FEEDBACK : receives
    RECIPE_VERSION ||--o{ COOKING_FEEDBACK : feedback_for
```

**핵심 기능:**
- 레시피북 CRUD
- 레시피 버전 관리 (최대 10개 버전 유지)
- 조리 피드백 수집
- 조리 이력 관리

### 3.4 AI Agent Domain

**목적:** 피드백 기반 레시피 보정, Q&A 응답, 자동 태깅

```mermaid
erDiagram
    ADJUSTMENT_REQUEST {
        uuid id PK
        uuid cookbook_recipe_id FK
        uuid feedback_id FK
        string status
        jsonb input_context
        jsonb output_result
        int processing_time_ms
        timestamp created_at
        timestamp completed_at
    }

    KNOWLEDGE_CHUNK {
        uuid id PK
        string source_type
        string source_id
        text content
        vector embedding
        jsonb metadata
        timestamp created_at
    }

    QA_SESSION {
        uuid id PK
        uuid user_id FK
        uuid recipe_id FK
        jsonb conversation_history
        timestamp created_at
        timestamp updated_at
    }

    ADJUSTMENT_REQUEST ||--o{ KNOWLEDGE_CHUNK : references
    QA_SESSION ||--o{ KNOWLEDGE_CHUNK : uses
```

**핵심 기능:**
- Adjustment Agent: 피드백 기반 레시피 자동 보정
- Q&A Agent: 조리 중 질문 응답
- Tagging Agent: 레시피 자동 분류
- RAG 기반 지식 검색

### 3.5 Analytics Domain

**목적:** 사용자 행동 추적, 서비스 메트릭, 비즈니스 인사이트

```mermaid
erDiagram
    EVENT {
        uuid id PK
        uuid user_id FK
        string event_type
        string event_category
        jsonb event_data
        string session_id
        string device_type
        timestamp created_at
    }

    USER_METRIC {
        uuid id PK
        uuid user_id FK
        date metric_date
        int recipes_viewed
        int recipes_cooked
        int feedbacks_given
        int ai_adjustments
        float avg_session_duration
    }

    RECIPE_METRIC {
        uuid id PK
        uuid recipe_id FK
        date metric_date
        int view_count
        int cook_count
        int save_count
        float avg_rating
        int feedback_count
    }

    USER ||--o{ EVENT : generates
    USER ||--o{ USER_METRIC : has
    RECIPE ||--o{ RECIPE_METRIC : tracked
```

**핵심 기능:**
- 실시간 이벤트 수집 (Kafka)
- 일별/주별/월별 집계
- 사용자 세그먼트 분석
- A/B 테스트 지원

---

## 4. 도메인 간 통신 패턴

### 4.1 동기 통신 (Sync)

| 호출자 | 피호출자 | 방식 | 용도 |
|--------|---------|------|------|
| Gateway | Recipe Service | REST | 레시피 검색/조회 |
| Gateway | User Service | REST | 인증/사용자 조회 |
| Gateway | Cookbook Service | REST | 레시피북 CRUD |
| Cookbook Service | Recipe Service | gRPC | 원본 레시피 조회 |
| AI Service | Knowledge DB | gRPC | 벡터 검색 |

### 4.2 비동기 통신 (Async)

```mermaid
flowchart LR
    subgraph Producers["이벤트 발행자"]
        P1[Recipe Service]
        P2[User Service]
        P3[Cookbook Service]
    end

    subgraph Kafka["Apache Kafka"]
        T1[recipe.events]
        T2[user.events]
        T3[cookbook.events]
        T4[feedback.events]
    end

    subgraph Consumers["이벤트 소비자"]
        C1[AI Agent Service]
        C2[Analytics Service]
        C3[Notification Service]
    end

    P1 --> T1
    P2 --> T2
    P3 --> T3
    P3 --> T4

    T1 --> C2
    T2 --> C2
    T3 --> C1
    T3 --> C2
    T4 --> C1
    T4 --> C2
    T4 --> C3
```

---

## 5. 도메인 이벤트 정의

### 5.1 주요 도메인 이벤트

| 이벤트 | 발행 도메인 | 소비 도메인 | 설명 |
|--------|-----------|-----------|------|
| `RecipeViewed` | Recipe | Analytics | 레시피 상세 조회 |
| `RecipeSaved` | Cookbook | Analytics | 레시피북에 저장 |
| `CookingStarted` | Cookbook | Analytics | 조리 시작 |
| `CookingCompleted` | Cookbook | Analytics, AI | 조리 완료 |
| `FeedbackSubmitted` | Cookbook | AI, Analytics | 피드백 제출 |
| `AdjustmentRequested` | Cookbook | AI | AI 보정 요청 |
| `AdjustmentCompleted` | AI | Cookbook, Analytics | AI 보정 완료 |
| `UserRegistered` | User | Analytics | 회원 가입 |
| `UserPreferenceUpdated` | User | AI | 취향 설정 변경 |

### 5.2 이벤트 스키마 예시

```typescript
// FeedbackSubmitted 이벤트
interface FeedbackSubmittedEvent {
  eventId: string;
  eventType: 'FeedbackSubmitted';
  timestamp: string;
  version: '1.0';

  payload: {
    userId: string;
    cookbookRecipeId: string;
    versionId: string;
    feedback: {
      tasteRating: number;      // 1-5
      difficultyRating: number; // 1-5
      feedbackText: string;
      adjustmentRequests: {
        category: 'taste' | 'portion' | 'difficulty' | 'ingredient';
        description: string;
      }[];
    };
  };

  metadata: {
    correlationId: string;
    causationId: string;
    userId: string;
  };
}
```

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|-----|------|----------|
| v1.0 | 2025.11.30 | 초기 문서 작성 |

---

> **다음 문서:** [5-1-2_SYSTEM.md](./5-1-2_SYSTEM.md) - 시스템 아키텍처

# 내시피(Naecipe) 운영 가이드

> 상위 문서: [5-1SERVICE_ARCHITECTURE.md](./5-1SERVICE_ARCHITECTURE.md)

---

## 1. 비용 최적화

### 1.1 AWS 비용 구조

```mermaid
flowchart TB
    subgraph Compute["컴퓨트 (45%)"]
        EKS[EKS 클러스터]
        EC2[EC2 인스턴스]
        LAMBDA[Lambda]
    end

    subgraph Database["데이터베이스 (30%)"]
        RDS[RDS PostgreSQL]
        REDIS[ElastiCache Redis]
    end

    subgraph Network["네트워크 (10%)"]
        NAT[NAT Gateway]
        ALB[ALB]
        CF[CloudFront]
    end

    subgraph Storage["스토리지 (10%)"]
        S3[S3]
        EBS[EBS]
    end

    subgraph Other["기타 (5%)"]
        MSK[MSK Kafka]
        SECRETS[Secrets Manager]
        CW[CloudWatch]
    end
```

### 1.2 월간 예상 비용 (Phase 1)

| 서비스 | 스펙 | 수량 | 월간 비용 (USD) |
|--------|------|------|----------------|
| **EKS Control Plane** | - | 1 | $73 |
| **EC2 (General)** | m6i.xlarge | 3 | $432 |
| **EC2 (AI)** | c6i.2xlarge | 2 | $490 |
| **RDS PostgreSQL** | db.r6g.xlarge | 5 | $1,450 |
| **ElastiCache Redis** | r6g.large | 6 | $648 |
| **MSK Kafka** | kafka.m5.large | 3 | $660 |
| **NAT Gateway** | - | 2 | $90 |
| **ALB** | - | 1 | $25 |
| **CloudFront** | 100GB/월 | 1 | $10 |
| **S3** | 500GB | 1 | $12 |
| **Secrets Manager** | 20 secrets | 1 | $8 |
| **CloudWatch** | - | 1 | $50 |
| **합계** | | | **~$3,948** |

### 1.3 비용 최적화 전략

```mermaid
flowchart LR
    subgraph Short["단기 (즉시)"]
        S1[Spot 인스턴스<br/>70% 절감]
        S2[Reserved Instances<br/>1년 30% 절감]
        S3[S3 Lifecycle<br/>30일 후 Glacier]
    end

    subgraph Medium["중기 (3-6개월)"]
        M1[Right-sizing<br/>사용량 기반 조정]
        M2[HPA 최적화<br/>야간 스케일 다운]
        M3[Cache 효율화<br/>히트율 95%+]
    end

    subgraph Long["장기 (6개월+)"]
        L1[Graviton 마이그레이션<br/>20% 절감]
        L2[Savings Plans<br/>3년 45% 절감]
        L3[Multi-AZ 최적화]
    end

    Short --> Medium
    Medium --> Long
```

### 1.4 FinOps 대시보드

| 메트릭 | 목표 | 현재 | 상태 |
|--------|------|------|------|
| **월간 비용** | < $4,000 | $3,948 | ✅ |
| **CPU 활용률** | > 60% | 45% | ⚠️ 개선 필요 |
| **메모리 활용률** | > 70% | 65% | ⚠️ |
| **Cache Hit Rate** | > 95% | 92% | ⚠️ |
| **Spot 비율** | > 50% | 0% | ❌ 적용 필요 |
| **일일 크롤링 레시피** | > 50 | - | 모니터링 필요 |

---

## 1.5 Crawler Bot 운영

### 크롤링 현황 모니터링

| 메트릭 | 목표 | 설명 |
|--------|------|------|
| **일일 신규 레시피** | > 50개 | 새로 수집된 레시피 수 |
| **중복률** | < 30% | 기존 레시피와 중복 비율 |
| **LLM 파싱 성공률** | > 95% | 레시피 추출 성공률 |
| **플랫폼별 차단율** | < 1% | Rate Limit 또는 차단 비율 |

### Crawler Bot 비용

| 항목 | 월간 비용 (USD) | 비고 |
|------|----------------|------|
| **OpenAI API (GPT-4)** | ~$200 | 레시피당 약 $0.02 |
| **YouTube Data API** | $0 | 무료 할당량 내 |
| **EC2 (t3.medium)** | $35 | 크롤러 실행 서버 |
| **합계** | ~$235 | |

---

## 2. 확장성 설계

### 2.1 수평 확장 전략

```mermaid
flowchart TB
    subgraph Current["현재 (Phase 1)"]
        C1[동시 사용자: 50,000]
        C2[RPS: 5,000]
        C3[DB: 5개 인스턴스]
    end

    subgraph Phase2["Phase 2 (6개월)"]
        P2A[동시 사용자: 200,000]
        P2B[RPS: 20,000]
        P2C[DB: Read Replica 추가]
    end

    subgraph Phase3["Phase 3 (12개월)"]
        P3A[동시 사용자: 500,000+]
        P3B[RPS: 50,000]
        P3C[DB: 샤딩]
    end

    Current --> Phase2
    Phase2 --> Phase3
```

### 2.2 데이터베이스 샤딩 전략

```mermaid
flowchart TB
    subgraph ShardingStrategy["샤딩 전략"]
        subgraph UserShard["User Shard (user_id 기반)"]
            US1[Shard 1<br/>user_id % 4 = 0]
            US2[Shard 2<br/>user_id % 4 = 1]
            US3[Shard 3<br/>user_id % 4 = 2]
            US4[Shard 4<br/>user_id % 4 = 3]
        end

        subgraph RecipeShard["Recipe Shard (Range 기반)"]
            RS1[Shard 1<br/>A-H]
            RS2[Shard 2<br/>I-P]
            RS3[Shard 3<br/>Q-Z]
        end
    end

    subgraph Router["Shard Router"]
        PROXY[ProxySQL / Vitess]
    end

    PROXY --> UserShard
    PROXY --> RecipeShard
```

### 2.3 확장 트리거

| 메트릭 | 임계값 | 액션 |
|--------|--------|------|
| CPU 사용률 | > 70% (5분) | Pod 1개 추가 |
| Memory 사용률 | > 80% (5분) | Pod 1개 추가 |
| RPS/Pod | > 1,000 | Pod 1개 추가 |
| Queue Length | > 1,000 | Consumer 추가 |
| DB Connections | > 80% | Connection Pool 확장 |
| Cache Hit Rate | < 90% | 캐시 크기 증가 |

---

## 3. 장애 복구 (DR)

### 3.1 DR 아키텍처

```mermaid
flowchart TB
    subgraph Primary["Primary Region (ap-northeast-2)"]
        subgraph PrimaryVPC["VPC"]
            PEKS[EKS Cluster]
            PRDS[(RDS Primary)]
            PREDIS[(ElastiCache)]
        end
        PS3[(S3 Primary)]
    end

    subgraph DR["DR Region (ap-northeast-1)"]
        subgraph DRVPC["VPC"]
            DREKS[EKS Cluster<br/>Standby]
            DRRDS[(RDS Read Replica)]
            DRREDIS[(ElastiCache<br/>Standby)]
        end
        DRS3[(S3 Replica)]
    end

    subgraph Global["Global"]
        R53[Route 53<br/>Health Check]
        GDB[Aurora Global Database]
    end

    PRDS -->|Async Replication| DRRDS
    PS3 -->|Cross-Region Replication| DRS3
    R53 --> Primary
    R53 -.->|Failover| DR
```

### 3.2 RTO/RPO 목표

| 시나리오 | RTO | RPO | 복구 전략 |
|---------|-----|-----|----------|
| **서비스 장애** | 5분 | 0 | 자동 재시작 |
| **AZ 장애** | 15분 | 0 | Multi-AZ Failover |
| **Region 장애** | 30분 | 5분 | DR Region 활성화 |
| **데이터 손상** | 1시간 | 5분 | Point-in-Time Recovery |
| **전체 재해** | 4시간 | 1시간 | 백업 복원 |

### 3.3 장애 복구 절차

```mermaid
flowchart TB
    START((장애 감지)) --> ASSESS{영향도 평가}

    ASSESS -->|Minor| AUTO[자동 복구<br/>Pod 재시작]
    ASSESS -->|Major| NOTIFY[On-Call 알림]

    NOTIFY --> ANALYZE[원인 분석]
    ANALYZE --> DECIDE{복구 방법 결정}

    DECIDE -->|AZ Failover| AZ[AZ 페일오버<br/>15분]
    DECIDE -->|Region Failover| REGION[DR 활성화<br/>30분]
    DECIDE -->|Data Recovery| DATA[PITR 복원<br/>1시간]

    AZ --> VERIFY[서비스 검증]
    REGION --> VERIFY
    DATA --> VERIFY

    VERIFY --> STABLE{안정화 확인}
    STABLE -->|Yes| POSTMORTEM[포스트모템]
    STABLE -->|No| ANALYZE

    POSTMORTEM --> END((종료))
    AUTO --> END
```

---

## 4. On-Call 운영

### 4.1 에스컬레이션 체계

```mermaid
flowchart TB
    subgraph L1["L1: 자동화 (0-5분)"]
        AUTO[자동 복구]
        ALERT[알림 발송]
    end

    subgraph L2["L2: On-Call (5-15분)"]
        ONCALL[On-Call 엔지니어]
        RUNBOOK[Runbook 실행]
    end

    subgraph L3["L3: 전문가 (15-30분)"]
        EXPERT[도메인 전문가]
        DEEP[심층 분석]
    end

    subgraph L4["L4: 경영진 (30분+)"]
        MGMT[경영진 보고]
        CRISIS[위기 관리]
    end

    L1 -->|해결 안됨| L2
    L2 -->|해결 안됨| L3
    L3 -->|심각| L4
```

### 4.2 On-Call 로테이션

| 주차 | Primary | Secondary | 시간대 |
|------|---------|-----------|--------|
| Week 1 | 개발자 A | 개발자 B | 24/7 |
| Week 2 | 개발자 B | 개발자 C | 24/7 |
| Week 3 | 개발자 C | 개발자 D | 24/7 |
| Week 4 | 개발자 D | 개발자 A | 24/7 |

### 4.3 Runbook 예시

```markdown
# Runbook: 높은 에러율 대응

## 트리거
- 알림: `HighErrorRate` (5xx > 1%)

## 즉시 확인 사항
1. Grafana 대시보드 확인
   - 어떤 서비스에서 에러 발생?
   - 에러 시작 시점?
   - 최근 배포 여부?

2. 로그 확인
   ```bash
   kubectl logs -n naecipe-prod -l app=<service> --tail=100
   ```

3. Pod 상태 확인
   ```bash
   kubectl get pods -n naecipe-prod -l app=<service>
   kubectl describe pod <pod-name> -n naecipe-prod
   ```

## 복구 절차

### 시나리오 1: Pod 장애
```bash
# Pod 재시작
kubectl rollout restart deployment/<service> -n naecipe-prod

# 롤백 (최근 배포 문제 시)
kubectl rollout undo deployment/<service> -n naecipe-prod
```

### 시나리오 2: DB 연결 문제
```bash
# Connection Pool 확인
kubectl exec -it <pod> -n naecipe-prod -- \
  python -c "from app.db import engine; print(engine.pool.status())"

# DB 상태 확인 (RDS Console)
```

### 시나리오 3: 외부 서비스 장애
- Circuit Breaker 확인
- 대체 서비스 활성화

## 에스컬레이션
- 15분 내 해결 안됨 → L3 호출
- 고객 영향 > 5% → 경영진 보고
```

### 4.4 Runbook: Crawler Bot 장애 대응

```markdown
# Runbook: Crawler Bot 장애 대응

## 트리거
- 알림: `CrawlerJobFailed` (CronJob 실패)
- 알림: `LowCrawlRate` (일일 크롤링 < 10개)

## 즉시 확인 사항
1. CronJob 상태 확인
   ```bash
   kubectl get cronjobs -n naecipe-crawler
   kubectl get jobs -n naecipe-crawler --sort-by=.metadata.creationTimestamp
   ```

2. 최근 Job 로그 확인
   ```bash
   kubectl logs job/recipe-crawler-youtube-xxxxx -n naecipe-crawler
   ```

3. 플랫폼 API 상태 확인
   - YouTube API 할당량 확인 (Google Cloud Console)
   - Instagram API 상태 확인

## 복구 절차

### 시나리오 1: API 할당량 초과
```bash
# 다음 날까지 대기 또는 할당량 증가 요청
# 임시로 다른 플랫폼 크롤링 실행
kubectl create job --from=cronjob/recipe-crawler-instagram manual-instagram -n naecipe-crawler
```

### 시나리오 2: LLM 파싱 실패율 증가
```bash
# 실패 로그 분석
kubectl logs job/recipe-crawler-youtube-xxxxx -n naecipe-crawler | grep "parse_error"

# 프롬프트 또는 모델 변경 필요 시 새 이미지 배포
```

### 시나리오 3: 플랫폼 차단
- robots.txt 변경 확인
- User-Agent 변경 검토
- 크롤링 간격 조정

## 수동 크롤링 실행
```bash
# 특정 채널만 크롤링
kubectl run manual-crawl --image=naecipe/recipe-crawler:latest \
  --restart=Never -n naecipe-crawler -- \
  python main.py --platform=youtube --channels="백종원의 요리비책" --max-recipes=20
```
```

---

## 5. 개발 환경 가이드

### 5.1 로컬 개발 환경

```mermaid
flowchart TB
    subgraph LocalDev["로컬 개발 환경"]
        subgraph DockerCompose["Docker Compose"]
            PG[(PostgreSQL)]
            REDIS[(Redis)]
            KAFKA[Kafka]
            ES[Elasticsearch]
        end

        subgraph Services["로컬 서비스"]
            GW[API Gateway<br/>:8000]
            RECIPE[Recipe Service<br/>:3001]
            USER[User Service<br/>:3002]
            AI[AI Agent<br/>:8001]
        end

        subgraph Frontend["프론트엔드"]
            NEXT[Next.js<br/>:3000]
        end
    end

    NEXT --> GW
    GW --> RECIPE
    GW --> USER
    RECIPE --> PG
    RECIPE --> REDIS
    USER --> PG
    AI --> KAFKA
```

### 5.2 Docker Compose 설정

```yaml
# docker-compose.yaml

version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: naecipe
      POSTGRES_PASSWORD: password
      POSTGRES_DB: naecipe
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U naecipe"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"

  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data

  # 로컬 서비스들
  recipe-service:
    build:
      context: ./services/recipe-service
      target: development
    ports:
      - "8001:8001"
    environment:
      ENVIRONMENT: development
      DATABASE_URL: postgresql://naecipe:password@postgres:5432/recipe
      REDIS_URL: redis://redis:6379
      KAFKA_BROKERS: kafka:9092
    volumes:
      - ./services/recipe-service:/app
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started

  # Recipe Crawler Bot (로컬 개발용)
  recipe-crawler:
    build:
      context: ./services/recipe-crawler
      target: development
    environment:
      ENVIRONMENT: development
      INGESTION_API_URL: http://recipe-service:8001/api/v1/ingestion
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      YOUTUBE_API_KEY: ${YOUTUBE_API_KEY}
    volumes:
      - ./services/recipe-crawler:/app
    depends_on:
      - recipe-service
    # 수동 실행: docker-compose run recipe-crawler --platform=youtube

volumes:
  postgres_data:
  redis_data:
  es_data:
```

### 5.3 개발 명령어

```bash
# 환경 설정
cp .env.example .env

# 가상 환경 생성 및 활성화
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Docker 서비스 시작
docker-compose up -d

# DB 마이그레이션
alembic upgrade head

# 개발 서버 시작
uvicorn app.main:app --reload --port 8001

# 테스트 실행
pytest

# 린트 & 포맷
ruff check .
ruff format .

# 빌드 (Docker 이미지)
docker build -t recipe-service .

# Crawler Bot 실행 (로컬)
docker-compose run recipe-crawler python main.py --platform=youtube --mode=once
docker-compose run recipe-crawler python main.py --platform=instagram --mode=once

# Crawler Bot 스케줄 모드 (백그라운드)
docker-compose up -d recipe-crawler
```

---

## 6. 상세 데이터 플로우

### 6.1 전체 시퀀스 다이어그램

```mermaid
sequenceDiagram
    autonumber
    participant U as 👤 User
    participant W as 🌐 Web
    participant GW as 🚪 Gateway
    participant RS as 📖 Recipe
    participant US as 👤 User
    participant CS as 📚 Cookbook
    participant K as 📨 Kafka
    participant AI as 🤖 AI Agent
    participant N as 🔔 Notification
    participant AN as 📊 Analytics

    Note over U, AN: 1️⃣ 인증 플로우
    U->>W: 로그인
    W->>GW: POST /auth/login
    GW->>US: 인증 요청
    US-->>GW: JWT 토큰
    GW-->>W: Set-Cookie
    US->>K: UserLoggedIn

    Note over U, AN: 2️⃣ 검색 플로우
    U->>W: 검색: "김치찌개"
    W->>GW: GET /recipes/search?q=김치찌개
    GW->>RS: 검색 요청
    RS->>RS: Elasticsearch 검색
    RS-->>GW: 검색 결과
    GW-->>W: 레시피 목록
    RS->>K: RecipeSearched

    Note over U, AN: 3️⃣ 상세 조회 플로우
    U->>W: 레시피 선택
    W->>GW: GET /recipes/{id}
    GW->>RS: 상세 조회
    RS->>RS: 캐시 확인
    RS-->>GW: 레시피 상세
    GW-->>W: 레시피 정보
    RS->>K: RecipeViewed

    Note over U, AN: 4️⃣ 레시피 저장 플로우
    U->>W: 레시피북에 저장
    W->>GW: POST /cookbooks/{id}/recipes
    GW->>CS: 레시피 저장
    CS->>CS: DB 저장
    CS-->>GW: 저장 완료
    GW-->>W: 성공 응답
    CS->>K: RecipeSaved

    Note over U, AN: 5️⃣ 조리 플로우
    U->>W: 조리 시작
    W->>GW: POST /cookbooks/{id}/recipes/{id}/cook
    GW->>CS: 조리 세션 시작
    CS-->>GW: 세션 ID
    GW-->>W: 조리 모드 진입
    CS->>K: CookingStarted

    loop 각 단계
        U->>W: 다음 단계
        W->>W: 로컬 상태 업데이트
    end

    U->>W: 조리 완료
    W->>GW: PUT /cookbooks/{id}/recipes/{id}/cook
    GW->>CS: 조리 완료 처리
    CS-->>GW: 완료
    CS->>K: CookingCompleted

    Note over U, AN: 6️⃣ 피드백 플로우
    U->>W: 피드백 입력
    W->>GW: POST /cookbooks/{id}/recipes/{id}/feedback
    GW->>CS: 피드백 저장
    CS->>CS: DB 저장
    CS-->>GW: 저장 완료
    GW-->>W: 접수 확인
    CS->>K: FeedbackSubmitted

    Note over U, AN: 7️⃣ AI 보정 플로우
    K->>AI: FeedbackSubmitted 수신
    AI->>CS: 레시피 & 피드백 조회
    CS-->>AI: 데이터
    AI->>AI: 피드백 분석
    AI->>AI: RAG 검색
    AI->>AI: 레시피 보정
    AI->>CS: 보정 결과 저장
    CS->>CS: 새 버전 생성
    AI->>K: AdjustmentCompleted

    Note over U, AN: 8️⃣ 알림 플로우
    K->>N: AdjustmentCompleted 수신
    N->>W: Push 알림
    W->>U: "레시피가 보정되었습니다"

    Note over U, AN: 📊 Analytics (비동기)
    K->>AN: 모든 이벤트 수신
    AN->>AN: 이벤트 집계
    AN->>AN: 메트릭 업데이트
```

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|-----|------|----------|
| v1.0 | 2025.11.30 | 초기 문서 작성 |

---

> **이전 문서:** [5-1-7_SECURITY.md](./5-1-7_SECURITY.md) - 보안 및 품질
> **다음 문서:** [5-1-9_ADR.md](./5-1-9_ADR.md) - 아키텍처 결정 기록

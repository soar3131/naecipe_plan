# 내시피(Naecipe) 종합 리팩토링 마스터 플랜 v2

작성일: 2026-02-19  
기준 문서: PRD, 2-1, 2-2, 3-1~3-3, 5-1-1~5-1-9

## 1. 목적
이 문서는 단순 정합성 점검이 아니라, **실제 개발/출시 가능한 문서 체계로 전환**하기 위한 리팩토링 실행 계획서다.

핵심 목표:
1. PRD-요구사항-설계-운영까지 추적 가능한 구조 확립
2. Recipe SSOT 중심 데이터 구조 확정
3. Agent/크롤러/검색/추천을 MVP와 확장 단계로 분리
4. 구현 착수 가능한 백로그와 주차별 산출물 확정

---

## 2. 근거 기반 핵심 이슈
1. `2-1REQUIREMENT.md`는 Phase1에서 크롤링을 별도 배치로 다루나, `5-1-2_SYSTEM.md`/`5-1-3_AI_AGENT.md`는 핵심 런타임 서비스처럼 기술되어 범위 불일치.
2. `5-1-6_INFRA.md` Deployment 예시가 Node 템플릿(`NODE_ENV`, 3001) 중심으로, FastAPI 주 스택 문서들과 충돌.
3. `5-1-8_OPERATIONS.md` 비용/DR/확장 가정이 MVP 대비 과도.
4. Agent 문서는 워크플로우 예시가 강하지만 운영 지표(Eval/비용/SLA/가드레일) 명세가 약함.

---

## 3. 실행 원칙
- 원칙 A: PRD 기능ID 없는 기술 도입 금지
- 원칙 B: SSOT 경계 정의 전 API/검색/추천 확장 금지
- 원칙 C: Agent 결과는 근거/버전/재현성 확보
- 원칙 D: 이벤트/인프라는 단계 도입 (MVP 단순화)
- 원칙 E: 모든 변경은 문서+결정로그(ADR)+실행백로그 동시 갱신

---

## 4. 20단계 리팩토링 실행

### Phase A. 범위/추적성 고정
1. PRD 기능ID 체계 확정 (`F-*`)
2. MVP 범위 동결 (`MVP_SCOPE_FREEZE.md`)
3. Traceability 매트릭스 작성 (`TRACEABILITY_MATRIX.md`)
4. 용어 사전 정규화 (용어 충돌 해소)
5. 문서 계층/참조 링크 재정렬

### Phase B. SSOT/데이터 파이프라인
6. Recipe SSOT 모델 확정 (`SSOT_SCHEMA_DRAFT.md`)
7. raw→canonical→variant 계보(lineage) 규칙 확정
8. 중복 판정 정책 엔진화 (exact/embedding/LLM)
9. Ingestion 계약(API + idempotency + DLQ) 정의
10. 검색 인덱싱/동기화 규칙 정의

### Phase C. Agent/Crawler 실전화
11. Adjustment Agent 입출력 계약 고정
12. Q&A Agent 근거표시/안전가드 정책 확정
13. Tagging ontology/승인 정책 정의
14. Agent 평가 체계 정의 (`AGENT_EVAL_PLAN.md`)
15. Crawler 실행 정책(스케줄/차단/재처리) 정식화

### Phase D. 아키텍처/운영 단계화
16. 서비스 경계 최소화(MVP 기준)
17. 이벤트 아키텍처 단계화 (`EVENT_ARCHITECTURE_STAGING.md`)
18. 인프라 단계화 (MVP/Scale)
19. 비용 모델 재산정 (MVP 운영 기준)
20. ADR/문서 세트 재승인 및 실행 백로그 배포

---

## 5. 완료 기준 (Definition of Done)
- 기능ID 기준으로 PRD→요구사항→화면→API→DB→운영 추적 가능
- SSOT 기준 테이블/버전/계보 규칙 확정
- Kafka/EKS 포함 도입 기술이 도입 시점 조건으로 명시
- Agent 품질/비용/지연 지표와 실패 대응 정책 문서화
- 12주 실행 백로그가 담당/산출물/검증기준 포함 상태

---

## 6. 즉시 산출물 목록
- MVP_SCOPE_FREEZE.md
- SSOT_SCHEMA_DRAFT.md
- TRACEABILITY_MATRIX.md
- EVENT_ARCHITECTURE_STAGING.md
- AGENT_EVAL_PLAN.md
- ADR 업데이트 (단계 도입 조건 반영)

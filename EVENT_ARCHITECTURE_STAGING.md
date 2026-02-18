# Event Architecture Staging Plan

작성일: 2026-02-19

## Stage 1 (MVP)
- 패턴: DB Outbox + 단일 큐(예: SQS/Rabbit/Redis Streams)
- 목표: 단순 운영, 재시도/멱등 확보
- 필수 이벤트:
  - feedback.submitted
  - recipe.adjustment.requested
  - recipe.adjustment.completed
  - recipe.saved

## Stage 2 (Scale)
- 조건 충족 시 Kafka/MSK 전환
- 조건 예시:
  1. 이벤트 처리량이 단일 큐 한계 도달
  2. 다중 소비자 독립 리플레이 필요
  3. 장기 보존 + 순서 보장이 핵심 요구

## 전환 시 필수 체크
- 이벤트 스키마 레지스트리 도입
- 토픽/파티션/리텐션 정책 확정
- 소비자 멱등성/재처리 테스트 통과

## 권장 결정
- Kafka는 MVP 필수가 아닌 “조건부 도입 기술”로 ADR 반영

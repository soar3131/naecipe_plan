# Recipe SSOT Schema Draft v1

작성일: 2026-02-19

## 1) 목적
레시피 데이터의 진실 원천을 분리해, 수집/정규화/보정/개인화를 계보로 추적한다.

## 2) 핵심 엔티티
- `recipe_raw`: 크롤링/수집 원문 저장
- `recipe_canonical`: 정규화된 기준 레시피
- `recipe_variant`: 보정/개인화/파생 버전
- `recipe_provenance`: 출처/수집시간/라이선스/신뢰도
- `recipe_dedup_judgement`: 중복 판정 근거 로그

## 3) 버전/계보 규칙
- raw -> canonical (N:1 가능)
- canonical -> variant (1:N)
- variant는 parent_recipe_id 필수
- 모든 변환은 `transformation_reason`, `agent_version` 기록

## 4) 품질 필드
- confidence_score
- completeness_score
- toxicity_or_policy_flag
- review_status (auto/pending/approved/rejected)

## 5) 최소 DDL 예시
```sql
create table recipe_canonical (
  id uuid primary key,
  title text not null,
  servings int,
  cooking_time_minutes int,
  difficulty text,
  canonical_hash text unique,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table recipe_variant (
  id uuid primary key,
  canonical_id uuid not null references recipe_canonical(id),
  parent_variant_id uuid null references recipe_variant(id),
  variant_type text not null check (variant_type in ('adjusted','personalized','editorial')),
  change_summary jsonb,
  agent_version text,
  created_by_user_id uuid,
  created_at timestamptz not null default now()
);
```

## 6) 운영 원칙
- 원본(raw) 삭제 금지 (법적 요청 시 비식별 마스킹)
- canonical 재생성 시 이전 버전 보존
- dedup 판정 로그는 감사용으로 보관

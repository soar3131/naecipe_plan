# Agent Evaluation & Guardrail Plan

작성일: 2026-02-19

## 대상
- Adjustment Agent
- Q&A Agent
- Tagging Agent

## 공통 평가 지표
1. 품질: 정답성/일관성/안전성
2. 성능: p95 지연시간
3. 비용: 요청당 평균 토큰/비용
4. 안정성: 실패율/재시도율

## Adjustment Agent
- 정확도: 보정안 수용률(사용자 저장/재사용)
- 안전성: 비현실 조리 지시 차단율
- 회귀테스트: 고정 샘플셋 100개

## Q&A Agent
- 근거성: 답변에 출처 포함 비율
- 불확실성 처리: 근거 부족 시 보수 응답률
- 금지영역: 의료/위험 조리법 가드레일

## Tagging Agent
- 태그 적합도 Precision/Recall
- 태그 온톨로지 이탈률

## 운영 정책
- 모델 변경 전 A/B + 오프라인 Eval 필수
- 비용 상한 초과 시 fallback 모델 전환
- 월별 품질 리포트 발행

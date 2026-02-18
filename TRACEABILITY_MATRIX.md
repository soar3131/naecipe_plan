# Traceability Matrix (Draft)

작성일: 2026-02-19

| Feature ID | PRD | Requirement | Feature Spec | API | DB/SSOT | Agent | Ops |
|---|---|---|---|---|---|---|---|
| F-CORE-SEARCH | PRD 5.1 | REQ-SEARCH-01~03 | H-01/H-02/S-01 | recipe search API | recipe_canonical index | Tagging(보조) | 검색 SLA |
| F-CORE-DETAIL | PRD 5.2 | REQ-DETAIL-01~03 | R-01~R-05 | recipe detail API | canonical + provenance | - | 캐시 정책 |
| F-CORE-FEEDBACK | PRD 5.5 | 조리기록/피드백 요구 | C-01/C-02 | feedback API | cooking_record | Adjustment trigger | 이벤트 수집 |
| F-CORE-ADJUST | PRD 5.5 | AI 보정 요구 | A-01/A-02 | adjustment API | recipe_variant | Adjustment Agent | Agent SLA |
| F-CORE-BOOK | PRD 5.4 | REQ-BOOK-01~02 | B-01/B-02 | cookbook API | cookbook + variant ref | - | 백업 정책 |
| F-CORE-QNA | PRD 5.6 | Q&A 요구 | Q-01/Q-02 | qna API | knowledge refs | Q&A Agent | 안전 정책 |

## 누락/보강 메모
- KPI 매핑 표를 별도 문서로 분리 필요
- 이벤트 스키마 문서와 연결 필요

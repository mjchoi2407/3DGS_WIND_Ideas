# 2026-08-14 01 아이디어 리뷰 수용·반박 지침

## Context

외부 아이디어 리뷰가 들어왔을 때 모든 지적을 기계적으로 수용하거나 기존 설계를 방어적으로 유지하지 않고,
현재 연구 목적과 증거에 따라 수용·부분 수용·반박·감수/보류를 구분하는 지속적 지침이 필요했다.

## Decisions

- 리뷰를 명령이 아니라 검증할 연구 입력으로 취급한다.
- 리뷰가 참조한 revision과 현재 canonical source를 먼저 대조해 stale 지적을 구분한다.
- 각 피드백에서 문제 진단의 타당성과 제안된 해결책의 타당성을 별도로 판정한다.
- 판정을 `수용 / 부분 수용 / 반박 / 감수·보류`로 명시하고 근거와 재검토 조건을 남긴다.
- 반박은 관련 수식, 물리 계약, 실험 또는 1차 자료로 뒷받침하고 판정을 뒤집을 수 있는 falsifier를 함께 둔다.
- 문서 수정은 사용자가 승인한 경우에만 수행하며, 수용 또는 반박 내용을 총평에만 두지 않고 관련 장과 downstream claim·실험 계약에 반영한다.
- 내부 sketch의 명시적 판단은 제출 manuscript에서 중립적인 설계 근거·비교·limitation으로 변환한다.

## Changed Files

- `AGENTS.md`
- `sessions/2026-08-14_01_idea_review_response_policy.md`

## Verification

- 새 규칙이 기존 read-only review 원칙, canonical source 규칙과 session tracking 규칙을 침범하지 않는지 확인했다.
- Markdown diff를 검토했고 `git diff --check`가 통과했다.

## Next

다음 아이디어 리뷰부터 session note에 항목별 `판정 / 근거 / 문서 영향 / 검증 또는 재검토 조건`을 사용한다.

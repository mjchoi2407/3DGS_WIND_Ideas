# Wind3DGS Ideas Instructions

Wind3DGS의 ideas 독립 저장소다. 공통 지침은 [`../AGENTS.md`](../AGENTS.md)를 적용한다.
상위 지침이 자동 로드되지 않은 하위 저장소 단독 실행에서도 직접 확인한다. 이미 확인한 내용은 반복 출력하지 않는다.
공통 언어·기록·Git·보안·LaTeX 규칙을 이 파일에 복제하지 않는다.

## Startup Protocol

- 공통 시작 절차에 따라 `README.md`의 현행 문서 링크와 `sessions/README.md`의 최신 요약을 확인한다.
- Framing/pivot은 sketch 관련 절, 구현·milestone은 master와 해당 R 절, novelty는 관련연구와 필요한 참고문헌만 읽는다.
- 외부 리뷰 대응에서는 `review_workflow.md`를 추가로 읽는다. LaTeX 작업은 해당 source와 owning build 절차를 확인한다.

## Editing Rules

- Preserve existing user files unless explicitly asked to reorganize them.
- 현재 방법의 authority는 `README.md`다. 파일명, 최근 session note 또는 `backup/`의 내용만 보고 현재 방향을 판단하지 않는다.
- 논문 아이디어가 실질적으로 변경되면 다음 순서로 전환한다.
  1. 이전 canonical source와 전달 산출물을 `backup/`에 보존한다.
  2. 새 sketch, bibliography, checklist, PDF와 delivery bundle을 필요한 범위에서 만들고 검증한다.
  3. 새 방법의 contract를 다시 통과하기 전까지 기존 구현·실험 완료 상태를 미검증으로 되돌리고 승계하지 않는다.
  4. `README.md`와 `.gitignore`의 canonical deliverable allowlist를 마지막에 함께 바꿔 현재 방향을 전환한다.
  5. 날짜가 있는 idea-side session note 하나를 남기고, 맞춰야 할 `code/` 또는 `experiments/` 문서를 식별한다.
- TeX 수정 시 공통 LaTeX 빌드 규칙을 적용하고 canonical PDF와 필요한 bundle을 갱신한다.
- Keep implementation details out of this repo unless they are pseudocode, design notes, or experiment requirements.

## Canonical 산출물

- 편집 가능한 TeX/Bib source와 `README.md`가 명시하고 `.gitignore`가 허용한 현재 PDF/delivery bundle만 추적한다.
- `backup/`의 기존 tracked 산출물은 과거 기록이지 현재 방법의 근거가 아니다. 이 파일들은 보존하되 새 preview PDF, 임의 ZIP 또는 반복 revision 사본을 계속 쌓지 않는다.
- 임시 LaTeX/build output은 ignored build directory 또는 임시 directory에 둔다. 기본적으로 배포하지 않는다.
- canonical 파일명이 바뀌면 `README.md`와 `.gitignore`를 함께 갱신하고 의도한 PDF/bundle만 새로 추적 가능한지 확인한다.
- TeX, Bib, bundle, log 또는 environment 파일에 secret을 넣지 않는다. `.env`와 `.env.*`는 제외하고 secret이 없는 `.env.example`만 추적할 수 있다.

## Session Tracking

- 공통 기록 기준을 따른다. 이 저장소의 고유 결정·변경·근거만 `sessions/`에 기록하고 다른 저장소의 상세 결과는 링크한다.
- `sessions/README.md`는 짧은 최신 진입점, 과거 목록은 `sessions/history.md`다. 중간 보고 원문은 기록하지 않는다.

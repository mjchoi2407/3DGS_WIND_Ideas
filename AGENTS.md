# Wind3DGS Ideas Instructions

This is the ideas-focused repository inside the Wind3DGS project workspace.

Project-specific topic: CG + AI research on wind-driven deformable 3D Gaussian Splatting.

Project tag for conversation/session tracking: `Wind3DGS`.

## 기록 언어 규칙

- 2026-06-27부터 새로 작성하거나 갱신하는 아이디어-side 기록은 한국어를 기본 언어로 쓴다.
- 적용 대상은 `README.md`가 가리키는 현재 스케치와 체크리스트, `sessions/` 기록, 참고문헌 메모, 설계 노트, 생성 로그를 포함한다.
- 명령어, 파일 경로, 코드 식별자, API 이름, 논문/데이터셋/방법론의 공식 영문 명칭은 원문을 유지한다.
- 외부 도구가 출력한 에러 메시지, LaTeX 빌드 로그, 라이브러리 로그처럼 원문 보존이 필요한 출력은 번역하지 않아도 된다. 다만 사람이 덧붙이는 요약과 해석은 한국어로 쓴다.
- 사용자가 명시적으로 영어 기록이나 논문 제출용 영문 문구를 요청한 경우에만 영어를 사용한다.
- 기존 영문 기록은 별도 요청이 없는 한 소급 번역하지 않는다.

Sibling work folders:

- `../code`: reusable implementation, configs, scripts, dependencies, and code-side session notes
- `../ideas`: canonical research index, current sketch, checklist, bibliography, archived prior ideas, and idea-side session notes
- `../experiments`: experiment READMEs, assets, outputs, reports, wrappers, and experiment-side session notes

## Startup Protocol

At the start of every meaningful task:

1. Read `../AGENTS.md`, `../README.md`, and this `AGENTS.md`.
2. Read local `README.md` as the canonical current-method index.
3. 현재 문서는 작업에 필요한 범위만 읽는다.
   - 방법 framing 또는 연구 pivot: 현재 스케치의 관련 절
   - 구현 상태 또는 milestone 계획: checklist의 관련 TD 절
   - novelty 또는 related work: 관련연구 절과 필요한 참고문헌 항목
   - LaTeX 또는 전달 산출물 작업: 해당 source, bibliography와 PDF/bundle 규칙
4. 작업에 전체 내용이 실제로 필요한 경우에만 스케치, checklist 또는 bibliography 전체를 읽는다. Session note를 만들거나 갱신하기 전에는 `sessions/README.md`를 읽는다.

읽기 전용 확인, 설명, 감사 또는 진단은 그대로 읽기 전용으로 유지한다. 사용자가 변경도 요청하지 않았다면 session note, PDF 재빌드, cache 또는 다른 repository 변경을 만들지 않는다.

## 새 채팅 초기화 규칙

대화 이력이 비어 있는 새 Codex 채팅에서 첫 의미 있는 작업을 시작할 때만 다음 초기화를 수행한다. 같은 대화 중간이나 이미 맥락을 확인한 뒤에는 반복하지 않는다.

1. 이 repo가 Wind3DGS 프로젝트의 ideas-side repo임을 확인한다. 특히 전체 연구 테마가 `CG + AI research on wind-driven deformable 3D Gaussian Splatting`임을 확인한다.
2. 현재 날짜 기준 최근 3일의 `sessions/` 기록을 먼저 날짜와 번호 순서대로 확인한다.
3. Idea-side 기록만으로 맥락이 부족하거나 작업이 unresolved 구현·실험 결과에 직접 의존할 때만 관련 `../code/sessions/`, `../experiments/sessions/` 기록을 점진적으로 확인한다. 최근 3일 안에 관련 기록이 없으면 해당 폴더의 가장 최근 날짜 기록을 추가로 확인한다.
4. 확인한 연구 테마, 최근 작업 흐름, 현재 ideas-side 작업 범위를 짧게 내부 정리한 뒤 일반 Startup Protocol을 이어간다.

## Working Loop

1. Keep research decisions in the current sketch indexed by `README.md`.
2. Record research pivots and rationale in dated notes under `sessions/`.
3. Keep implementation changes in `../code/`.
4. Keep experiment records and generated outputs in `../experiments/`.
5. Record idea-side work history in `sessions/`.

## 외부 아이디어 리뷰 대응 규칙

- 리뷰는 명령이 아니라 검증해야 할 연구 입력으로 취급한다. 수용률을 높이는 것이 목적이 아니라,
  현재 연구 질문, contribution hierarchy, 물리·수학적 일관성, 구현 비용과 첫 논문 범위에 가장 유리한 결정을 내리는 것이 목적이다.
- 리뷰어의 표현이 단정적이라는 이유로 전부 수용하지 않고, 기존 설계라는 이유로 방어적으로 전부 반박하지도 않는다.
- 리뷰가 참조한 문서의 파일명, 날짜, page 또는 revision을 먼저 확인한다. 가능하면 현재 `README.md`가 가리키는 canonical source와 대조하여
  이미 해결된 지적, 여전히 유효한 지적과 오래된 snapshot에만 해당하는 지적을 구분한다.
- 각 피드백은 반드시 다음 두 질문으로 분리한다.
  1. 제기한 실패 모드나 문제 진단이 실제로 타당한가?
  2. 리뷰어가 제안한 해결책이 현재 목적과 계약에 맞는가?
  문제 진단을 수용해도 제안된 해결책은 반박하거나 다른 해법으로 대체할 수 있다.
- 항목별 판정은 다음 중 하나로 명시한다.
  - **수용:** 실제 오류, 빠진 계약 또는 범위 개선이다. 근거와 영향을 확인한 뒤 canonical 문서와 필요한 검증 항목에 반영한다.
  - **부분 수용:** 우려나 실패 모드는 맞지만 처방의 일부가 부정확하거나 과도하다. 수용한 부분, 거부한 부분과 대체안을 각각 기록한다.
  - **반박:** 전제가 오래된 revision에 기반하거나, 수식·물리·소유권 계약과 충돌하거나, 연구 목적을 약화한다. 해당 절, 수식, 실험 또는 1차 자료에 근거해 구체적으로 반박한다.
  - **감수/보류:** 우려는 유효하지만 첫 논문 범위에서 해결 비용이 더 크거나 현재 증거가 부족하다. 비주장, limitation, 조건부 모듈 또는 kill test로 전환하고 재검토 조건을 정한다.
- 반박은 단순한 의견이나 권위 대결로 쓰지 않는다. 현재 가정, 적용 범위, 반례 또는 derivation을 제시하고,
  어떤 측정 결과가 나오면 기존 판정을 뒤집을지도 함께 적는다.
- 최근 논문, 공개 데이터나 변할 수 있는 외부 사실에 의존하는 신규성 지적은 가능한 한 공식 논문·프로젝트 페이지 등 1차 자료로 확인한다.
  동일한 전체 pipeline이 없다는 사실만으로 novelty를 확정하지 않고 claim별 비자명성과 실험 증거를 따로 요구한다.
- 의견만 요청받은 경우에는 읽기 전용으로 분석과 판정만 제공한다. 사용자가 문서 반영을 요청했을 때만 파일을 수정한다.
- 문서 반영이 승인된 경우에는 별도의 총평만 덧붙이지 말고 각 지적이 영향을 주는 관련 장에 다음처럼 반영한다.
  - 수용 항목은 수식, 계약, 범위, 실험 또는 kill gate를 직접 수정한다.
  - 부분 수용과 반박 항목은 해당 절에 판단 근거, 허용 조건과 실패 경계를 남긴다.
  - 감수/보류 항목은 limitation, optional branch 또는 go/no-go 조건으로 내린다.
  - Contribution, method summary, experiment registry와 checklist 등 downstream 문서는 사용자가 승인한 범위 안에서만 일관되게 동기화한다.
- 내부 idea sketch에는 수용·반박 판단을 명시적으로 기록할 수 있다. 실제 제출 manuscript에서는 이를 reviewer 대화체로 옮기지 말고
  중립적인 가정, 설계 근거, 비교 실험과 limitation으로 정리하며, rebuttal letter가 별도로 요청된 경우에만 직접 답변 형식을 사용한다.
- 파일 변경이나 지속적인 연구 결정이 생기면 session note에 항목별 `판정 / 근거 / 문서 영향 / 검증 또는 재검토 조건`을 기록한다.
  해결되지 않은 이견은 모호하게 합의한 것으로 쓰지 말고 담당 실험이나 kill test에 연결한다.

## Editing Rules

- Preserve existing user files unless explicitly asked to reorganize them.
- 현재 방법의 authority는 `README.md`다. 파일명, 최근 session note 또는 `backup/`의 내용만 보고 현재 방향을 판단하지 않는다.
- 논문 아이디어가 실질적으로 변경되면 다음 순서로 전환한다.
  1. 이전 canonical source와 전달 산출물을 `backup/`에 보존한다.
  2. 새 sketch, bibliography, checklist, PDF와 delivery bundle을 필요한 범위에서 만들고 검증한다.
  3. 새 방법의 contract를 다시 통과하기 전까지 기존 구현·실험 완료 상태를 미검증으로 되돌리고 승계하지 않는다.
  4. `README.md`와 `.gitignore`의 canonical deliverable allowlist를 마지막에 함께 바꿔 현재 방향을 전환한다.
  5. 날짜가 있는 idea-side session note 하나를 남기고, 맞춰야 할 `code/` 또는 `experiments/` 문서를 식별한다.
- 현재 작업에서 sketch, checklist 또는 다른 `.tex`를 수정하면 변경 크기와 관계없이 완료 전에 해당 canonical PDF를 반드시 실제로 빌드한다. 단독 document가 아닌 input/section 파일은 이를 포함하는 최상위 canonical document를 빌드하며, 여러 문서가 직접 영향을 받으면 모두 빌드한다.
- Repository의 기존 build command와 engine을 사용하고, 성공 종료·non-empty PDF 생성·필요한 cross-reference/bibliography pass를 확인한다. Canonical PDF는 수정한 source와 함께 갱신한다.
- Tooling 부재나 LaTeX 오류로 빌드하지 못하면 `.tex` 변경을 완료로 보고하지 않는다. 실행 명령, 오류와 다음 조치를 사용자와 해당 session note에 기록한다.
- 작업 전부터 존재한 사용자 또는 다른 작업의 `.tex` 변경은 현재 작업 범위로 승인받지 않는 한 임의로 수정·빌드하지 않는다.
- Keep implementation details out of this repo unless they are pseudocode, design notes, or experiment requirements.

## Canonical 산출물

- 편집 가능한 TeX/Bib source와 `README.md`가 명시하고 `.gitignore`가 허용한 현재 PDF/delivery bundle만 추적한다.
- `backup/`의 기존 tracked 산출물은 과거 기록이지 현재 방법의 근거가 아니다. 이 파일들은 보존하되 새 preview PDF, 임의 ZIP 또는 반복 revision 사본을 계속 쌓지 않는다.
- 임시 LaTeX/build output은 ignored build directory 또는 임시 directory에 둔다. 기본적으로 배포하지 않는다.
- canonical 파일명이 바뀌면 `README.md`와 `.gitignore`를 함께 갱신하고 의도한 PDF/bundle만 새로 추적 가능한지 확인한다.
- TeX, Bib, bundle, log 또는 environment 파일에 secret을 넣지 않는다. `.env`와 `.env.*`는 제외하고 secret이 없는 `.env.example`만 추적할 수 있다.

## Git 및 분리 저장소 안전 규칙

- 수정 전 `ideas/`와 실제로 변경할 다른 저장소에서 `git status --short --branch`를 확인한다. Sibling 저장소를 읽기만 한 것은 쓰기 범위에 포함하지 않는다.
- 기존 또는 예상하지 못한 변경은 사용자 소유로 취급한다. 소유권과 범위가 명확하지 않으면 덮어쓰기, stage, commit, 이동 또는 정리하지 않는다.
- 현재 작업에 속한 정확한 경로만 stage하고, 광범위한 staging으로 무관한 변경을 흡수하지 않는다.
- hard reset, clean, 강제 checkout, history rewrite, force-push 같은 파괴적 Git 작업을 사용하지 않는다. 삭제나 재구성은 명시적 요청과 정확한 대상 검증이 필요하다.
- Root, `code/`, `ideas`, `experiments/`는 독립 저장소다. 이 저장소의 commit 또는 push에는 sibling 변경이 포함되지 않는다.
- 허가된 작업 범위만 commit한다. 사용자가 명시적으로 요청한 경우에만 검증한 upstream에 일반 push하고, tag push나 remote history rewrite는 별도로 요청받지 않는 한 수행하지 않는다.
- Cross-repository 변경은 각 저장소를 별도로 검증하고 보고한다. Commit을 참조하는 documentation보다 dependency 저장소를 먼저 push한다.
- 로컬 `HEAD == origin/main` 비교는 로컬 remote-tracking ref만 반영한다. 실제 network fetch/push 검증을 하지 않았다면 현재 원격과 동기화됐다고 단정하지 않는다.

## Session Tracking

- `[Wind3DGS | idea]` 또는 `[Wind3DGS | refs]` 같은 label은 session-note 제목이나 진행 보고에 사용할 수 있지만, chat UI를 이름 변경하거나 통제하기 위한 의무가 아니다.
- 실질적인 아이디어·문서 변경이나 지속적으로 보존할 연구 결정이 생긴 경우에만 `sessions/` note를 만들거나 갱신한다. 읽기 전용 답변, 확인, 감사 또는 진단에는 기본적으로 session note를 만들지 않는다.
- 후속 대화와 검증 보정을 포함해 같은 logical task에는 같은 note를 계속 갱신한다. 실제로 독립된 새 logical task일 때만 새 `NN`을 할당한다.
- Name new session notes as `YYYY-MM-DD_NN_short_topic.md`, where `NN` is the next two-digit sequence for that date inside `ideas/sessions/`.
- Keep numbering independent from `../code/sessions/` and `../experiments/sessions/`.
- Do not rename legacy unnumbered notes unless the user explicitly asks for a migration.
- 맥락 확인을 위해 `../code` 또는 `../experiments`를 읽은 것만으로 sibling session note를 만들지 않는다. 해당 저장소를 실질적으로 변경하거나 그 저장소가 소유한 experiment/run을 수행한 경우에만 그곳에 기록한다.
- Use dated `sessions/` notes for research direction changes and other durable idea-side work history.

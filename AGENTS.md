# Wind3DGS Ideas Instructions

This is the ideas-focused repository inside the Wind3DGS project workspace.

Project-specific topic: CG + AI research on wind-driven deformable 3D Gaussian Splatting.

Project tag for conversation/session tracking: `Wind3DGS`.

## 기록 언어 규칙

- 2026-06-27부터 새로 작성하거나 갱신하는 아이디어-side 기록은 한국어를 기본 언어로 쓴다.
- 적용 대상은 `idea_sketch.tex`, `changelog.md`, `sessions/` 기록, 참고문헌 메모, 설계 노트, 생성 로그를 포함한다.
- 명령어, 파일 경로, 코드 식별자, API 이름, 논문/데이터셋/방법론의 공식 영문 명칭은 원문을 유지한다.
- 외부 도구가 출력한 에러 메시지, LaTeX 빌드 로그, 라이브러리 로그처럼 원문 보존이 필요한 출력은 번역하지 않아도 된다. 다만 사람이 덧붙이는 요약과 해석은 한국어로 쓴다.
- 사용자가 명시적으로 영어 기록이나 논문 제출용 영문 문구를 요청한 경우에만 영어를 사용한다.
- 기존 영문 기록은 별도 요청이 없는 한 소급 번역하지 않는다.

Sibling work folders:

- `../code`: reusable implementation, configs, scripts, dependencies, and code-side session notes
- `../ideas`: idea sketches, checklists, bibliography, research direction changelog, and idea-side session notes
- `../experiments`: experiment READMEs, assets, outputs, reports, wrappers, and experiment-side session notes

## Startup Protocol

At the start of every meaningful task:

1. Read `../RESEARCH_PROJECT_GUIDE.md` if available from the workspace root.
2. Read this `AGENTS.md`.
3. Check local idea-side records:
   - `idea_sketch.tex`
   - `changelog.md`
   - `refs.bib`
   - `sessions/README.md`

## 새 채팅 초기화 규칙

대화 이력이 비어 있는 새 Codex 채팅에서 첫 의미 있는 작업을 시작할 때만 다음 초기화를 수행한다. 같은 대화 중간이나 이미 맥락을 확인한 뒤에는 반복하지 않는다.

1. 이 repo가 Wind3DGS 프로젝트의 ideas-side repo임을 확인한다. 특히 전체 연구 테마가 `CG + AI research on wind-driven deformable 3D Gaussian Splatting`임을 확인한다.
2. 현재 날짜 기준 최근 3일의 session 기록을 확인한다. 우선 `sessions/`를 보고, 작업이 구현이나 실험과 연결되면 `../code/sessions/`, `../experiments/sessions/`도 날짜와 번호 순서대로 훑는다.
3. 최근 3일 안에 session 기록이 없거나 작업 맥락이 부족하면, 관련 session 폴더에서 가장 최근 날짜의 기록을 추가로 확인한다.
4. 확인한 연구 테마, 최근 작업 흐름, 현재 ideas-side 작업 범위를 짧게 내부 정리한 뒤 일반 Startup Protocol을 이어간다.

## Working Loop

1. Keep early research decisions in `idea_sketch.tex`.
2. Record research pivots in `changelog.md`.
3. Keep implementation changes in `../code/`.
4. Keep experiment records and generated outputs in `../experiments/`.
5. Record idea-side work history in `sessions/`.

## Editing Rules

- Preserve existing user files unless explicitly asked to reorganize them.
- When a paper idea changes, update `idea_sketch.tex` and add a dated entry to `changelog.md`.
- After editing `idea_sketch.tex`, build `idea_sketch.pdf` to verify the LaTeX output. If the build fails or LaTeX tooling is unavailable, report the reason and record it in the session note.
- Keep implementation details out of this repo unless they are pseudocode, design notes, or experiment requirements.

## Session Tracking

- Start substantial new conversations with a prefix like `[Wind3DGS | idea]` or `[Wind3DGS | refs]`.
- At the end of meaningful idea work, create or update a note under `sessions/`.
- Name new session notes as `YYYY-MM-DD_NN_short_topic.md`, where `NN` is the next two-digit sequence for that date inside `ideas/sessions/`.
- Keep numbering independent from `../code/sessions/` and `../experiments/sessions/`.
- Do not rename legacy unnumbered notes unless the user explicitly asks for a migration.
- Use `changelog.md` for research direction changes, not ordinary conversation history.

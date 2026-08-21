# 2026-08-13 10 아이디어 저장소 작업 지침 보강

## 목적

새 Global--Local 연구 방향을 구현하는 동안 canonical 문서, 백업, 생성 산출물과 Git 작업 경계가 혼동되지 않도록 idea-side 지침을 보강한다.

## 결정

- 읽기 전용 확인·감사에는 session note를 만들지 않고, 같은 logical task는 기존 note를 계속 갱신한다.
- `ideas/README.md`만 현재 연구 방향의 authority로 사용하며 `backup/`은 현행 근거로 사용하지 않는다.
- 연구 pivot은 이전 canonical 보존, 새 문서 검증, 상태 비승계, README와 allowlist 전환 순서로 수행한다.
- 현재 README가 가리키는 PDF/bundle만 새로 추적하고 임시 PDF와 임의 revision ZIP은 제외한다.
- dirty worktree의 기존 변경을 사용자 소유로 취급하고 정확한 경로만 stage하며, destructive Git과 암묵적 push를 금지한다.
- 작업에 필요한 현재 문서 부분만 점진적으로 읽고, 전체 문서 로드는 필요한 경우로 제한한다.
- 현재 작업에서 `.tex`를 수정하면 변경 크기와 관계없이 해당 canonical PDF를 실제로 빌드하며, 빌드 실패 시 완료로 보고하지 않는다.

## 변경 파일

- `AGENTS.md`
- `.gitignore`
- `README.md`
- `sessions/README.md`
- `sessions/2026-08-13_10_workflow_policy_update.md`

## 검증

- Markdown과 ignore 규칙의 diff를 확인했다.
- canonical PDF/bundle은 추적 가능하고 임의 PDF/ZIP과 `.env`, `.env.local`은 제외되며 `.env.example`은 추적 가능한 것을 확인했다.
- 기존 backup PDF/ZIP 10개가 계속 tracked 상태임을 확인했다.
- `git diff --check`가 통과했다.

## 범위 메모

작업 도중 현재 아이디어 TeX에 이 작업이 만들지 않은 동시 변경이 나타났다. 해당 변경은 사용자 소유로 간주해 수정·stage·재빌드하지 않았으며, 이 지침 보강의 변경 범위와 검증 결과에서 분리했다.

## 다음 단계

실제 연구 pivot 시 README 포인터와 `.gitignore` canonical allowlist를 같은 변경으로 갱신한다.

# 2026-08-21 01 Minimal V0 개념 해설 작성

## Context

현재 canonical idea sketch는 구현 계약과 수학적 경계를 상세히 닫았지만, 전체 아이디어를 저자 관점에서
쉽게 이해하고 비판적으로 검토하기에는 수식과 typed contract의 밀도가 높다. Canonical authority를
복제하거나 바꾸지 않고, 문제 정의부터 Setup, frame runtime, Gaussian transport, 연구 Gate와 scope까지를
개념적 언어로 연결하는 파생 Markdown 문서를 새로 작성했다.

## Decisions

- 개념 설명은 `바람 외력 / 구조 복원 / 관성·감쇠`의 세 효과로 구성했다. Global과 Local은 서로 다른
  힘이 아니라 같은 힘에 반응하는 거시적·국소 보완 움직임 공간이라고 명시했다.
- Static GS에서 sparse anchor scaffold를 만드는 이유, topology distillation이 예측하는 것과 analytic
  decoder/runtime이 담당하는 것을 분리했다.
- Setup과 runtime을 나누고, full-anchor aero, deterministic Local selection, Active/Decay, single coupled
  solve, anchor-to-Gaussian transport의 frame 흐름을 수식 없이 설명했다.
- Rest area ownership과 runtime safe area, selector score와 실제 force consumption, Global/Local force
  projection과 double counting의 차이를 별도 설명했다.
- MC1/MC2/SYS, Gate A/B/C, 사전 등록 attempt와 failure denominator, 공식 deferred 8개, 명시적 비주장과
  Gate 전 scope freeze를 이해·검토 질문에 연결했다.
- `oracle_flat_strip_3x7` 같은 code-side 예시나 새 backend/stage는 해설 문서의 요구사항으로 만들지 않았다.
- Method equation과 claim은 canonical sketch가 소유하고, checklist는 파생 실행 문서, 새 concept guide는
  비권위 이해·검토 문서라는 순서를 README와 문서 첫머리에 명시했다.

## Changed Files

- `wind3dgs_minimal_v0_concept_guide_2026-08-21.md`
  - 1분 요약, 전체 흐름도, 용어 표, 힘/운동 역할, Setup/runtime, 예시, 기여/Gate, 근사/비주장,
    deferred 조건과 저자 검토 질문을 포함한다.
- `README.md`
  - `이해·검토용 개념 해설` 절에서 새 문서를 연결하고 권위 경계를 명시했다.

Canonical sketch/checklist TeX와 PDF는 이번 작업에서 수정하지 않았다.

## Verification

- Canonical sketch의 topology/structural package, Global/Local basis, aero, selector/state, coupled solve,
  transport, Gate/deferred 절과 문장 단위로 대조했다.
- 세 개의 독립 읽기 감사에서 setup→runtime→render 흐름, 힘과 Global/Local의 구분, MC1/MC2/SYS,
  Gate와 scope-control을 점검했다.
- 감사 중 발견한 다음 표현을 보정했다.
  - 외부 바람장이 0이어도 물체가 움직이면 relative-wind drag가 남을 수 있음
  - Active는 score만이 아니라 dwell/admission/capacity policy를 통과한 실제 pre-solve 집합임
  - SYS scaling evidence와 final Gate C의 end-to-end Pareto는 서로 다른 증거임
  - Renderer transport는 arbitrary thickness deformation을 주장하지 않음
  - In-envelope failure는 성공 결과만 남기기 위해 denominator에서 제거하지 않음
- 최종 독립 감사 판정은 blocker/major 0의 PASS다. 새 solver, network, state class, runtime stage 또는
  asset class 요구가 생기지 않았음을 확인했다.
- 상대 링크 대상 두 TeX가 존재하고 non-empty이며, Markdown/README에 trailing whitespace,
  conflict marker 또는 stale 금지 표현이 없음을 확인했다. `git diff --check`도 통과했다.
- `.tex`를 수정하지 않았으므로 LaTeX/PDF build는 수행하지 않았다.

Final SHA-256:

- concept guide: `18fff70f1492c6ed40f6545ce54654dafa990a2ffd16432455d69a7716e4b804`
- ideas README: `e42dcd4384247a71dad519f5fe59802d587cf95db82723ca886a344b0e4706b7`
- referenced canonical sketch TeX: `e54f6610be3f4a142b351069ddfb29492bfef2bc01e4ec8198afe537fe1b3a3e`
- referenced checklist TeX: `07e702f04bb290310216ef893a34a902ad1971244c763827958cbbde7b538458`

## Next

- 이 문서의 17절 검토 질문을 사용해 MC1/MC2 중 반드시 지킬 기여와 허용할 시각적 근사를 저자가
  다시 판단한다.
- 구현이나 실험이 시작되면 새 결과를 concept guide의 사실처럼 소급 반영하지 않고 canonical
  sketch/checklist와 code/experiments 기록에서 먼저 판정한다.

이번 작업은 ideas-side 설명 문서와 index만 변경했다. Code/experiment 구현, 실행 결과 또는 완료
상태를 생성하거나 승계하지 않았다.

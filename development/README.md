# Wind3DGS R0--R7 Development Plans

이 폴더는 현재 *Response-Distilled Global--Local Wind Dynamics* 아이디어를 실제 구현과 실험으로 옮기기 위한
파트별 실행 문서를 관리한다. 각 문서는 standalone XeLaTeX source이며 공통 표현은
[`shared_preamble.tex`](shared_preamble.tex)을 사용한다.

## Authority와 진행 소유권

- Method equation, 연구 범위와 claim의 최종 authority는 상위 canonical idea sketch인
  [`../3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex`](../3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex)다.
- Canonical implementation checklist인
  [`../implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex`](../implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex)는
  R0--R7 순서와 Gate A--D의 완료 기준을 정의한다.
- 이 폴더의 각 R 문서는 해당 파트의 구현 진행 상태, Open Design Decision, fixture 결과와 code/experiment artifact link를 소유한다.
- 동일 progress를 README나 다른 R 문서에 중복 기록하지 않는다. 이 README는 master index와 dependency만 소유한다.
- Part 문서가 sketch와 충돌하면 sketch가 우선한다. 지속적인 method decision은 sketch/checklist를 먼저 갱신한 뒤 관련 part 문서에 내려 쓴다.

## Master roadmap

| Part | 문서 | 핵심 목적 | 진입/종료 경계 |
|---|---|---|---|
| R0 | [`TeX`](r0_contract_and_schema.tex) / [`PDF`](r0_contract_and_schema.pdf) | Typed I/O, SI/unit, package schema, visibility와 failure denominator 동결 | 모든 후속 구현의 선행 계약 |
| R1 | [`TeX`](r1_teacher_probe_oracle.tex) / [`PDF`](r1_teacher_probe_oracle.pdf) | Teacher convergence, common probe, traction/ZOH, transport와 network-free oracle | Gate A\(_T\) prerequisite의 reference 경로 확보 |
| R2 | [`TeX`](r2_single_case_global_overfit.tex) / [`PDF`](r2_single_case_global_overfit.pdf) | Single-case Global package overfit과 long rollout | Network/package/evaluator 기본 가설 확인 |
| R3 | [`TeX`](r3_multi_resplat_measure.tex) / [`PDF`](r3_multi_resplat_measure.pdf) | Independent multi-resplat measure/response consistency와 latent ablation | Gate A\(_M\), Gate A\(_L\) 판정 및 fallback 선택 |
| R4 | [`TeX`](r4_heldout_global.tex) / [`PDF`](r4_heldout_global.pdf) | Narrow-domain object-disjoint held-out Global response | Gate B 판정과 Global checkpoint 동결 |
| R5 | [`TeX`](r5_local_residual.tex) / [`PDF`](r5_local_residual.pdf) | Residual atlas, always-on Local 및 Local angular/normal branch | Gate C 판정; 실패 시 Global-only |
| R6 | [`TeX`](r6_conditional_local_runtime.tex) / [`PDF`](r6_conditional_local_runtime.pdf) | Analytic selector, persistent state, translation/angular fade와 fixed-budget runtime | Gate D 판정; 실패 시 always-on Local |
| R7 | [`TeX`](r7_renderer_and_paper_evidence.tex) / [`PDF`](r7_renderer_and_paper_evidence.pdf) | Frozen transport-chain 재현, optional SH appearance와 paper evidence hardening | 통과한 Gate에 맞는 최종 claim/evidence package |

실행 순서는 기본적으로 R0에서 R7까지다. 실패 routing은 module을 늘리는 신호가 아니라 claim과 다음 작업 범위를
줄이는 신호다. R1/R2가 실패하면 dataset이나 Local을 늘리지 않고, R3의 measure/latent 실패는 각각 fixed-family 또는
direct-set fallback으로 분리한다. R4 실패는 amortized learned-response pivot을 막고, R5 실패는 Global-only,
R6 실패는 always-on Local로 축소한다.

## 파트 문서 공통 구조

각 문서는 다음 항목을 같은 순서로 유지한다.

1. 목적과 소유권
2. 진입 입력
3. 동결 계약
4. 구현 체크리스트
5. Open Design Decisions
6. 산출물
7. Fixture와 metric
8. 종료 조건과 실패 경로

Open Design Decision은 아직 method authority가 아니다. 후보, 허용 경계와 판정 fixture를 기록하고,
결정이 method identity나 claim을 바꾸면 canonical sketch/checklist에 먼저 반영한다.

## R1 문서 안에서 구현 근거를 갱신하는 방법

[R1 TeX](r1_teacher_probe_oracle.tex) / [PDF](r1_teacher_probe_oracle.pdf) 하나에 실행 명세와 구현·실험 근거를 둔다.
별도 구현 기록의 전체 내용을 R1의 기존 8개 절에 통합했다.

- 수식·계약 변경은 해당 본문 식을 직접 교체하고 변경 이유·허용 범위를 인접 설명과 TeX 주석으로 남긴다.
- 구현 상태는 4절, 현행 개발 후보와 이전 식의 교체 근거는 5절, 산출물·재현 명령·hash는 6절에 둔다.
- 정량 결과는 7절, 최종 채택 여부와 남은 조건은 8절에서 관리한다.
- 실패한 식은 실패 원인을 설명하는 위치에 당시 적용 범위를 표시해 보존하고, 현행 채택 식으로 병기하지 않는다.
- 개발 후보의 식을 문서에 반영한 사실만으로 canonical R1 acceptance를 통과시킨 것으로 해석하지 않는다.

## 진행 상태 기록 규칙

각 part 문서에서 항목 상태는 다음 의미로 사용한다.

- `Pending`: 선행 입력 또는 구현이 아직 준비되지 않음
- `In progress`: owning code/experiment branch와 실행 artifact가 식별됨
- `Pass`: frozen fixture, raw result, command, commit과 hash가 연결됨
- `Fail`: fixed denominator에서 실패했고 정해진 routing을 적용함
- `Deferred`: core Gate와 분리된 optional branch이며 재검토 조건이 기록됨

체크박스만 표시하고 근거를 생략하지 않는다. `Pass`는 code/experiment repository의 commit, config/package/run hash,
재현 command와 report link를 함께 가질 때만 인정한다. 기존 explicit-scaffold 결과나 이전 milestone 상태를 새 R 문서의
완료 근거로 자동 승계하지 않는다.

## 변경 규칙

- 수식, I/O 의미, unit, visibility, runtime stage, Gate 또는 claim이 바뀌면 canonical sketch가 먼저 바뀐다.
- 실행 순서, schema field, fixture와 artifact 이름 같은 구현 계약은 관련 R 문서와 canonical checklist를 함께 맞춘다.
- 탐색 중인 hyperparameter와 후보는 Open Design Decisions에 남기고 test 결과를 본 뒤 소급 동결하지 않는다.
- 한 part의 결정이 downstream package identity를 바꾸면 영향을 받는 후속 part의 기존 `Pass`를 재검증 대상으로 돌린다.
- 각 part는 자신의 진행 내용만 소유한다. Cross-part dependency는 이 README의 roadmap과 입력 hash로 연결한다.
- 구현은 `../../code`, 실험 run/output/report는 `../../experiments`, 연구 방법과 본 개발 계획은 `..`가 소유한다.

## XeLaTeX 사용

각 source는 ideas repository root에서 다음 형태로 독립 빌드한다. `-cd`가 해당 source가 있는
`development/`로 이동해 `shared_preamble.tex`를 찾고, PDF도 같은 폴더에 생성한다.

```bash
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  development/r6_conditional_local_runtime.tex
```

최상위 master checklist는 ideas repository root에서 별도로 빌드한다.

R0--R7 PDF는 각 TeX와 함께 추적하는 current development deliverable이다. Part TeX를 수정하면 해당 PDF를,
[`shared_preamble.tex`](shared_preamble.tex)을 수정하면 R0–R7 PDF 8개를 모두 다시 빌드한다. `.aux`, `.log`, `.toc` 등
중간산출물은 추적하지 않으며, master 전달 bundle에는 master TeX/PDF, 이 README, shared preamble와
R0--R7 TeX/PDF를 상대 경로를 보존해 함께 넣는다.

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

각 R 문서는 **표지 → 장별 개발 체크리스트 1쪽 → 목차 → 본문** 구조다.
처음 구조가 필요할 때만 이 진입점을 확인하고, 작업 맥락이 충분하면 해당 계약·검증 절만 읽는다.
Master roadmap도 표지 다음에 R0–R7 단계별 종료 체크를 제공한다.
장/단계 이름을 클릭하면 해당 본문 또는 개발 PDF로 이동한다.

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

작업 정리 요청에서도 관련 R 문서의 상태·근거 동기화를 생략하지 않는다. 적용 절차는 [공통 지침](../../AGENTS.md#정리-요청과-r-문서-동기화)을 따른다.

## R1 문서 안에서 구현 근거를 갱신하는 방법

[R1 TeX](r1_teacher_probe_oracle.tex) / [PDF](r1_teacher_probe_oracle.pdf) 하나에 실행 명세와 구현·실험 근거를 둔다.
별도 구현 기록의 전체 내용을 R1의 기존 8개 절에 통합했다.

- 수식·계약 변경은 해당 본문 식을 직접 교체하고 변경 이유·허용 범위를 인접 설명과 TeX 주석으로 남긴다.
- 구현 상태는 4절, 현행 후보·교체 근거는 5절에 둔다. 6절은 산출물과 소유 experiment report/manifest를 연결하고 필요한 대표 재현 명령만 둔다.
- 대표 정량 결과와 적용 조건은 7절, 최종 채택·남은 조건은 8절에서 관리한다. 실행별 전체 수치·raw/hash는 experiment report/manifest가 소유한다.
- 실패한 식은 실패 원인을 설명하는 위치에 당시 적용 범위를 표시해 보존하고, 현행 채택 식으로 병기하지 않는다.
- 개발 후보의 식을 문서에 반영한 사실만으로 canonical R1 acceptance를 통과시킨 것으로 해석하지 않는다.

## 갱신 시점과 중복 방지

- 실행 중 프레임 수·대기·중간 보고를 TeX에 넣지 않는다. 진행 판단은 짧은 session 현재 상태에서 관리한다.
- 확정 결과·구현 계약·채택 경계가 바뀌면 해당 R 문서의 관련 절을 갱신한다. Sketch는 수식·방법/claim 변화, master는 단계/Gate 변화가 있을 때만 갱신한다.
- README는 포인터만 관리한다. 상세 실행 결과를 sketch/master/R 문서/session에 전량 복제하지 않는다.
- 기존 R1의 통합 근거는 이번 운영 변경으로 삭제하지 않는다. 관련 후속 작업에서 필요한 요약과 소유 report 링크로 정리한다.
- TeX를 수정한 경우에는 변경 크기와 관계없이 공통 PDF 빌드 규칙을 적용한다.

## 진행 상태 기록 규칙

작업 상태 복구와 기록 시점은 [공통 필수 기록 규칙](../../AGENTS.md#간결한-작업-기록)을 따른다. Note에는 이 문서의 관련 절 또는 R source의 정확한 `label`을 연결하고, 전체 R 문서를 반복해서 읽도록 요구하지 않는다.

표지 다음 체크리스트는 각 문서 본문의 8개 장과 일대일로 연결한다. 문서를 작성했거나 일부 기능만
구현한 상태는 완료 체크하지 않는다. 해당 장의 구현·필수 검증·산출물 근거와 종료 조건을 충족한 뒤
완료 처리한다. 단계별 종료 체크는 master가, 장별 종료 체크는 각 R 문서가 소유한다.

- `\DevProgressRow{0}{...}{진행/대기}{...}`의 첫 인자를 `1`로 바꾸면 완료 체크와 합계가 함께 갱신된다.
- 체크리스트 날짜는 기본2026-09-09다. 현행 상태를 갱신한 문서는 shared preamble 입력 뒤 `\renewcommand{\DevChecklistDate}{YYYY-MM-DD}`로 기준일을 명시한다. 날짜 갱신 자체는 완료 체크나 검증 통과를 뜻하지 않는다.
- 완료 처리할 때 본문의 실제 report/run/hash와 판단 근거를 먼저 갱신하고 상태·남은 작업도 수정한다.
- 장 전체의 종료 체크와 이미 검증된 개발 구현/국소 fixture를 구분한다.
  R1의 개발 성과는 같은 페이지의 별도 성과 목록에 체크해 표시한다.
  이는 canonical R1 acceptance나 학습 적격성을 뜻하지 않는다.
- 변경한 문서의 PDF를 실제 빌드하고 master bundle의 해당 source/PDF를 동기화한다.
  공통 페이지 macro가 있는 `shared_preamble.tex`를 바꾸면 R0–R7과 master PDF를 모두 빌드한다.
- 체크 수는 개발 시간이나 전체 작업량의 백분율로 해석하지 않는다.

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
[`shared_preamble.tex`](shared_preamble.tex)을 수정하면 이를 포함하는 R0–R7 PDF 8개와 master PDF를 모두 다시 빌드한다. `.aux`, `.log`, `.toc` 등
중간산출물은 추적하지 않으며, master 전달 bundle에는 master TeX/PDF, 이 README, shared preamble와
R0--R7 TeX/PDF를 상대 경로를 보존해 함께 넣는다.

# Wind3DGS Ideas

Wind3DGS의 연구 framing, 방법 명세, 참고문헌과 아이디어 작업 기록을 관리한다.

## 현재 아이디어 스케치

현재 작업본의 제목은 *Response-Distilled Global--Local Wind Dynamics for Static 3D Gaussian Thin Surfaces*다.
Training-only mesh simulation을 privileged teacher로 사용하지만, target inference에서는 static 3DGS,
metric/material/attachment와 prescribed wind만 받는다. Patch 기반 normalized latent token과 내부 relation에서
absolute deformation이 아닌 passive Global/Local response package를 예측하고, 필요한 Local residual만
fixed budget으로 실행하는 방향이다.

- LaTeX: `3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex`
- 참고문헌: `refs_response_distilled_global_local_wind_dynamics.bib`
- PDF: `3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.pdf`
- 전달 bundle: `3dgs_response_distilled_global_local_wind_dynamics_2026-08-22_bundle.zip`

정확한 claim은 `fully meshless`가 아니라 **target-mesh-free inference without a persistent physical adjacency graph**다.
Mesh vertex/connectivity와 mesh--GS correspondence는 teacher simulation과 학습 loss에만 사용한다.

## 현재 구현 체크리스트

새 방향의 완료 상태는 다음 파생 체크리스트에서 R0--R7과 Gate A--D로 관리한다.

- LaTeX: `implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex`
- PDF: `implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.pdf`
- 전달 bundle: `implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22_bundle.zip`

Method equation과 claim의 authority는 current sketch이며 checklist는 stable label을 참조하는 실행 문서다.

## 방향 전환 경계

이전 Minimal V0는 target static GS에서 oriented anchor, edge/triplet relation과 structural/reduced package를
구성하는 explicit-scaffold 방법이었다. 해당 canonical sketch/checklist/Bib/PDF/bundle과 concept guide는
`backup/minimal_v0_before_latent_anchor_response_2026-08-22/`에 원래 파일명으로 보존한다.
이 archive는 복구 가능한 baseline/설계 기록이며 새 방법의 요구사항이나 완료 근거가 아니다.

더 오래된 방향은 다음 archive에 보존한다.

- `backup/full_design_before_minimal_v0_2026-08-15/`: Minimal V0 이전 full design
- `backup/previous_idea_sketches_before_2026-08-13/`: 2026-08-13 이전 idea sketch
- `backup/review_materials_through_2026-08-21/`: 이전 방법 수정에 사용한 review provenance
- `backup/local_workspace_cleanup_2026-08-21/`: ignored build/download metadata의 로컬 보조 백업

## 구현·실험 상태

기존 `../code`와 `../experiments`의 `TD##`, topology/scaffold, fixed-Hessian PD와 Global/Local solver 경로는
이전 방향의 legacy/support 또는 비교 baseline이다. 새 R0--R7 contract를 통과하지 않은 기존 결과를
현재 learned-response 방법의 evidence로 승계하지 않는다.

이번 아이디어 전환에서는 `code/`와 `experiments/`를 수정하지 않았다. 다음 구현 작업은 먼저
teacher/student visibility, response-package schema와 source-object split을 동결하고, 기존 loader/teacher/renderer 중
재사용 가능한 부분만 명시적으로 인수해야 한다.

## Canonical 산출물 정책

- 이 README가 가리키는 sketch, bibliography, checklist와 각 PDF/bundle만 현행 방법의 canonical 산출물이다.
- `backup/` 파일과 historical session은 당시 방향의 provenance이며 current authority가 아니다.
- `ideas/` 최상위에는 정책 파일, current sketch/checklist 산출물과 필요한 현재 companion만 둔다.
- `.gitignore`는 current PDF/bundle과 명시적으로 동결한 archive PDF만 exact path로 허용한다.
- 임시 LaTeX build output, preview PDF와 임의 revision ZIP은 추적하지 않는다.
- 연구 방향 전환 시 이전 canonical을 먼저 checksum과 함께 보존하고, 새 문서/PDF/bundle을 검증한 뒤 포인터를 바꾼다.

## Project-Internal Split

- `../code`: 재사용 구현, 설정, 스크립트와 code-side session
- `../ideas`: 아이디어 스케치, 연구 framing, 참고문헌과 idea-side session
- `../experiments`: 실험 자산, 출력, 보고서와 experiment-side session

## 기록 위치

- 아이디어 대화와 작업 이력은 `sessions/`에 기록한다.
- 구현 작업은 `../code`에 둔다.
- 실험 기록과 출력은 `../experiments`에 둔다.

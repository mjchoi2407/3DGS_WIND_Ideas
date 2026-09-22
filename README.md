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
Mesh vertex/connectivity와 mesh--GS correspondence는 teacher simulation 및 training/evaluation fixture에만 사용한다.

## 현재 구현 체크리스트

새 방향의 전체 순서, 의존성과 Gate A--D routing은 짧은 master roadmap에서 관리한다.
개발 세부 내용은 `development/` 아래 R0--R7 독립 문서로 나누며, 실제 개발 중에는 현재 파트 문서만
갱신한다. 아직 설계가 필요한 선택은 각 문서의 Open Design Decisions에 두고 구현자가 암묵적으로 확정하지 않는다.

- LaTeX: `implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex`
- PDF: `implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.pdf`
- 전달 bundle: `implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22_bundle.zip`
- 파트별 index와 갱신 규칙: `development/README.md`
- 파트 문서: `development/r0_contract_and_schema.tex`부터
  `development/r7_renderer_and_paper_evidence.tex`까지의 standalone TeX/PDF 8쌍

Master와 각 R 문서의 표지 다음에는 단계별/장별 개발 체크리스트를 둔다.
완료 체크, 진행·대기 상태와 본문 링크로 확인하며, 부분적인 개발 진단 통과는 장 전체 완료와 구분한다.
체크와 근거·PDF를 함께 갱신하는 방법은 [`development/README.md`](development/README.md)에 있다.

Method equation과 claim의 authority는 current sketch다. Master roadmap은 stage/claim routing을,
각 part 문서는 구현 계약, 미결정 설계, fixture, 산출물과 종료 근거를 소유한다.

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

현행 연구 경계는 **물리 수식 구현·검증 후 학습데이터 생성**이다. 유한 회전 shell 경로를 사용하며,
개발 진단 통과를 R1 전체 채택이나 학습 적격성으로 승계하지 않는다.

2026-09-11 문서 감사에서30fps 사용자 목표, Teacher 실행 경로 선택과 후속 실패/미확정 결과를 반영했다.
반영 범위와 검증 한계는 [문서 감사 기록](sessions/2026-09-09_01_teacher_evidence_integration.md#현재-상태)을 따른다.

- [연구 결정 요약](sessions/README.md): 확정 계약과 문서 반영 상태.
- [R1 TeX](development/r1_teacher_probe_oracle.tex) / [PDF](development/r1_teacher_probe_oracle.pdf): 수식·구현 계약·대표 검증 결과·채택 경계.
- [실험 최신 요약](../experiments/sessions/README.md): 약한 바람, 4배 바람, GPU 성능의 결과와 미완료 범위.
- [구현 최신 요약](../code/sessions/README.md): 구현 상태와 다음 선택.

실행별 설정·명령·원본/hash·전체 수치는 해당 experiment report를 참조한다. 이 index에 진행 이력과
동일 수치를 복제하지 않는다. 기존 TD##/M## 결과는 현행 R0–R7 계약을 통과하기 전까지 legacy/support다.

## Canonical 산출물 정책

- 이 README가 가리키는 sketch, bibliography, master roadmap, `development/`의 R0--R7 TeX/PDF와
  두 전달 bundle만 현행 방법의 canonical 산출물이다.
- `backup/` 파일과 historical session은 당시 방향의 provenance이며 current authority가 아니다.
- `ideas/` 최상위에는 정책 파일과 current sketch/master 산출물을 두고, 파트별 개발 문서는
  `development/`에 둔다.
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

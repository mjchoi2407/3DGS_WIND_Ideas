# Wind3DGS Ideas

Wind3DGS의 연구 framing, 방법 명세, 참고문헌과 아이디어 작업 기록을 관리한다.

## 현재 아이디어 스케치

현재 작업본의 제목은 *Topology-Distilled, Error-Triggered Global--Local Wind Dynamics for Static 3D Gaussian Thin Surfaces*다.

- LaTeX: `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.tex`
- 참고문헌: `refs_topology_distilled_error_triggered_wind_dynamics.bib`
- PDF: `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.pdf`
- 전달 bundle: `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13_bundle.zip`

## 현재 구현 체크리스트

새 방법의 구현 상태는 다음 독립 체크리스트에서 관리한다.

- LaTeX: `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.tex`
- PDF: `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.pdf`
- 전달 bundle: `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13_bundle.zip`

이전 체크리스트의 완료 상태는 승계하지 않는다. 기존 static GS loader, viewer와 transport prototype은 새 schema와 P0 contract를 다시 통과한 뒤에만 현재 완료 상태로 인정한다.

2026-08-13 이전 아이디어 스케치와 관련 산출물은 `backup/previous_idea_sketches_before_2026-08-13/`에 원래 이름으로 보존한다.

## Project-Internal Split

- `../code`: 재사용 구현, 설정, 스크립트와 code-side session
- `../ideas`: 아이디어 스케치, 연구 framing, 참고문헌과 idea-side session
- `../experiments`: 실험 자산, 출력, 보고서와 experiment-side session

## 기록 위치

- 아이디어 대화와 작업 이력은 `sessions/`에 기록한다.
- 구현 작업은 `../code`에 둔다.
- 실험 기록과 출력은 `../experiments`에 둔다.

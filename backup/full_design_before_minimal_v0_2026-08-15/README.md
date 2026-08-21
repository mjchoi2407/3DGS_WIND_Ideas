# 2026-08-15 Minimal V0 전환 전 full design archive

이 폴더는 scope-control 검토에 따라 `Topology-Distilled, Error-Triggered Global--Local
Wind Dynamics`의 114쪽 full design을 Minimal V0로 축소하기 직전 상태로 보존한다.

## 보존 범위

- `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.tex`
- `refs_topology_distilled_error_triggered_wind_dynamics.bib`
- `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.pdf`
- `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13_bundle.zip`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.tex`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.pdf`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13_bundle.zip`

이 archive에는 당시 검토한 objective normal-curvature backend, support-restricted
KKT/Schur, hard/free/compliant reaction ledger, same-frame corrector, confidence calibration과
조건부 H/F/G 설계가 포함된다. 이 내용은 후속 연구 후보와 rationale이지 Minimal V0의 구현
요구사항이나 완료 증거가 아니다.

이 archive를 만든 2026-08-15 당시에는 상위 `ideas/README.md`가 가리키던 Minimal V0
sketch, bibliography와 checklist가 canonical source of truth였다. 현재 방법은 항상 최신
`ideas/README.md`의 canonical 링크를 따른다. 필요 시 이 archive의 모듈은 core benchmark에서
명백한 visual artifact, instability 또는 measured quality--latency 개선 필요가 확인된 뒤에만
개별적으로 재검토한다.

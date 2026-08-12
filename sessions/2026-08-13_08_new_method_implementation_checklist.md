# 2026-08-13 08 새 방법 구현 체크리스트

## 목적

백업된 기존 `implementation_checklist.md/.tex`의 상태 표기, 자동 검증 루프, milestone 완료 기준과 시각 스타일을 참고해 오늘 확정한 `Topology-Distilled, Error-Triggered Global--Local Wind Dynamics`용 새 구현 체크리스트를 작성했다.

기존 체크리스트 파일과 backup 내용은 수정하지 않았다. 이전 proxy/SH-residual 방향의 완료 상태도 새 문서로 승계하지 않았다.

## 새 파일

- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.tex`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.pdf`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13_bundle.zip`

## 구조

- 상태 표기 `[ ]`, `[~]`, `[x]`, `[!]`, `[K]`
- 자동 `구현 -> 검증 -> 수정 -> 재검증` 프로토콜
- 새 experiment tag `TD00`--`TD14`
- Runtime input/output, force/state owner ledger
- P0 implementation kill gate 13개
- Runtime Module 1--14와 milestone mapping
- 공통 artifact registry
- TD00--TD14별 선행 조건, 입력, 구현 항목, API, 검증, 완료 기준
- 학습 curriculum과 loss registry
- 직접/조건부 baseline 구현 registry
- metric, profiling, 통계 계약
- artifact/run directory layout
- Open Decisions와 결정 시한
- 즉시 실행 순서와 완료 정의

## 핵심 구현 계약

논리 번호와 실제 계산 dependency를 분리했다.

`1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 10 -> 9 -> 8 -> 11 -> 12 -> 13 -> 14`

Patch force proposal을 먼저 overlap-aware하게 조립하고 Global complement를 적용한 뒤 하나의 Local physical system을 적분한다. Patch별 integrate-then-average는 kill gate다.

추가로 다음 두 모호점을 구현 전 blocker로 명시했다.

1. Local generalized missing-force label과 patch nodal-force network target의 차원 변환
2. Global predictor가 이미 소비한 structural cross coupling과 corrector feedback의 delta 정의

## 이전 구현의 처리

기존 static GS loader, camera loader, synthetic fixtures, debug viewer, position/covariance transport, procedural wind preview는 `재사용 후보`로만 기록했다. 새 typed schema, affine transport와 E0/P0 unit tests를 통과하기 전에는 완료로 인정하지 않는다.

기존 mandatory SH residual은 승계하지 않았다. RSH는 TD06/E2 aerodynamic representation ablation으로만 남겼다.

## 검증

- LaTeX source 1,957줄
- XeLaTeX/latexmk 빌드 성공
- 최종 A4 PDF 33쪽
- LaTeX error, undefined reference, overfull box 없음
- 표지, owner/kill-gate, E0, aero/RSH, gate/state-machine, curriculum, 최종 handoff 대표 페이지 시각 확인
- 전달 bundle 압축 무결성 검사

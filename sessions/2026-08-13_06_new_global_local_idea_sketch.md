# 2026-08-13 06 새 Global--Local wind dynamics 아이디어 스케치

## 목적

오늘 논의한 `Global physics + error-triggered Local missing-force correction` 방향을 기존 문서의 revision이 아닌 새 독립 아이디어 문서로 정리했다. 기존 `idea_sketch.tex`, 이전 날짜의 bundle, changelog는 이번 작업에서 수정하지 않았다.

## 새 문서 제목과 파일

- 제목: *Topology-Distilled, Error-Triggered Global--Local Wind Dynamics for Static 3D Gaussian Thin Surfaces*
- LaTeX: `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.tex`
- 참고문헌: `refs_topology_distilled_error_triggered_wind_dynamics.bib`
- 빌드 PDF: `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.pdf`
- 전달 bundle: `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13_bundle.zip`

## 핵심 결정

1. Global은 current-surface quasi-steady aerodynamic load와 object-level reduced structural dynamics를 항상 물리식으로 적분한다.
2. Gate는 현재 변형 크기가 아니라 Local correction을 켰을 때 앞으로 줄일 수 있는 오차 대비 비용을 예측한다.
3. Local network는 위치나 최종 운동을 직접 출력하지 않고, 간이 물리가 놓친 missing force만 제안한다.
4. Overlap patch의 force proposal을 먼저 보존적으로 조립한 뒤 하나의 complementary local physical state를 적분한다.
5. Force target의 중복과 Global/Local 운동공간의 중복을 서로 다른 문제로 분리한다.
6. Local correction으로 바뀐 구조 반력과 current geometry aerodynamic load의 차이를 Global corrector에 되먹인다.
7. Gate가 꺼진 patch의 learned kernel은 실제 skip하되 기존 Local energy는 reset하지 않고 물리적으로 감쇠시킨다.
8. RSH는 mainline에서 제거하고 방향 표현/압축 ablation으로만 남긴다.

## 문서 구성

- Training-only teacher, topology/operator distillation, runtime mesh-free contract
- Gate와 missing-force label 생성 절차
- Runtime Module 1--14의 목적, solver 종류, 입력, 연산, 출력, 다음 전달값, 검증 조건
- Global/Local force와 motion ownership, overlap conservation, two-way feedback
- 학습 loss와 curriculum, runtime pseudocode, 복잡도 및 break-even 조건
- 관련 논문별 유사점, 차이점, 직접 baseline 여부, 필요한 비교 실험
- E0--E10 독립 비교 실험 대장과 전체 ablation registry
- Generalization, stress test, 평가 지표, 통계, go/no-go 및 P0 kill gate

## 검증

- `latexmk -xelatex -interaction=nonstopmode -halt-on-error` 빌드 성공
- 최종 PDF 64쪽, A4
- undefined reference/citation 없음
- overfull box 없음
- 대표 페이지에서 14단계 표, 관련연구 표, 비교 실험 대장, ablation 표, 참고문헌 배치를 시각 확인
- 소스 3,834줄, 전용 BibTeX 201줄

## 다음 작업

문서의 P0 순서대로 deterministic force ownership/assembly contract를 먼저 단위 테스트한 뒤, Global-only MVP, Oracle-gated Local, learned gate 순으로 구현한다. 초기 반증 실험에서는 RSH-free direct current-surface aerodynamic force를 기본형으로 두고 RSH 계열은 동일 budget의 비교군으로만 평가한다.

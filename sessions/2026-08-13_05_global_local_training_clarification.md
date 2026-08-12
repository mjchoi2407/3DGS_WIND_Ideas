# 2026-08-13 05 global/local 역할과 학습데이터 명확화

## 결론

Global은 알려진 surface aero와 reduced structural dynamics로 큰 운동을 계산한다. Local도 최종 운동을 neural network가 직접 생성하지 않는다. Network는 global/analytic model이 놓친 `missing local force`만 예측하고, patch별 작은 mass--stiffness--damping system이 이를 적분하여 local displacement와 velocity를 만든다.

`global/local 중복 힘 제거`라는 표현은 다음 두 채널로 구분해야 한다.

- local detail state는 global modal span과 mass-orthogonal하게 제한하여 중복 운동을 제거한다.
- local reaction과 corrected-aero의 global component는 제거하지 않고 global generalized force로 되먹임한다. 이것이 실제 양방향 coupling이다.

## 수정된 runtime 순서

1. 이전 global/local 상태로 현재 표면을 복원한다.
2. current surface와 prescribed wind에서 analytic aerodynamic force를 계산한다.
3. reduced structural dynamics로 global predictor를 적분한다.
4. learned gate가 global-only 미래 patch error/uncertainty를 예측한다.
5. active patch에서만 learned model이 missing local force를 예측한다.
6. patch-level reduced structural dynamics가 local state를 적분한다.
7. local detail을 global modal complement에 제한한다.
8. overlapping patches를 연속적·보존적으로 조립한다.
9. local reaction과 corrected-geometry aerodynamic delta를 global force로 되먹이고 global corrector를 푼다.
10. 최종 anchors/Gaussians를 갱신하고 global/local/gate state를 다음 frame에 저장한다.

Gate가 off인 patch는 missing-force network를 skip한다. 이미 남아 있는 local energy는 저렴한 local damping integration으로 감쇠시킨다.

## Gate 학습데이터

1. 다양한 object, material, attachment, spatial wind/gust에 대해 high-fidelity thin-shell FEM/MPM와 필요시 CFD/LBM teacher trajectory를 생성한다.
2. 같은 초기 상태와 wind로 global-only model을 rollout한다.
3. 각 patch에서 teacher와 global-only의 미래 위치, 속도, normal/curvature, strain 또는 aerodynamic-load 차이를 측정한다.
4. local solver를 켰을 때 줄일 수 있는 오차와 추가 비용을 평가하여 continuous benefit/error score를 만든다.
5. runtime에서 관측 가능한 global state, geometry, wind, history만 입력으로 사용해 이 score를 예측하도록 gate를 학습한다.

단순 현재 deformation magnitude가 아니라 `앞으로 local correction이 실제로 얼마나 유익한가`가 target이다.

## Missing-force 학습데이터

Teacher deformation을 global state와 global-complement local state로 분해한다. Local state의 teacher displacement, velocity, acceleration과 patch mass/stiffness/damping을 이용해 그 운동을 만들기 위해 필요했던 힘을 inverse dynamics로 구한다. 여기서 analytic aero와 known structural force가 이미 설명한 부분을 빼면 missing-force target이 된다.

Teacher가 force breakdown을 직접 기록하면 이를 사용한다. 위치 trajectory만 있으면 acceleration finite difference가 noisy하므로 smoothing과 inverse-dynamics consistency가 필요하다.

## 왜 local에 network를 쓰는가

Local의 inertia, elasticity, restoring, damping처럼 알려진 물리는 물리식으로 푼다. Network는 다음처럼 저렴한 local model이 놓친 항만 근사한다.

- global/local basis truncation
- large-deformation thin-shell nonlinearities
- imperfect topology/material distillation
- quasi-steady aero가 놓친 flutter/wake/history effect

이 항들을 analytic하게 풀려면 active patch에 full thin-shell FEM/MPM 또는 fluid solver가 다시 필요해진다. 정확도와 비용이 허용되면 local을 완전한 물리 solver로 만드는 것도 가능하며 필수적으로 neural이어야 하는 것은 아니다.

## Global도 neural로 풀 수 있는가

가능하지만 권장 기본형은 `physics base + learned generalized-force correction`이다. Global reduced solve는 이미 매우 작고 저렴하다. 이를 direct next-state network로 대체하면 wind-stop recovery, resonance, long rollout stability, material editability, unseen wind generalization을 데이터로 다시 학습해야 한다.

비교 baseline은 다음 세 가지가 적절하다.

- physics global: analytic aero + reduced structural integration
- hybrid global: physics global + learned missing generalized force
- direct neural global: current state/wind에서 next modal state를 직접 예측

Hybrid가 필요한 경우에도 structural restoring/damping은 항상 physics base로 유지한다.

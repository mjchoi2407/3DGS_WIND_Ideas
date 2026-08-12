# 2026-08-13 04 runtime 14단계별 solver 할당

## 권장 원칙

Runtime 전체를 neural response로 만들지 않는다. 학습 네트워크는 `global-only 오차 gate`와 `active patch missing-force predictor`에 집중한다.

- 상태/기하 복원: deterministic kinematics
- 공기력: prescribed wind에 대한 quasi-steady surface aerodynamic law
- global force 축약: conservative empirical cubature와 Galerkin projection
- global/local 시간 반응: reduced second-order structural dynamics의 implicit integration
- patch 선택과 누락 힘: learned models
- 중복 제거와 patch 조립: mass-orthogonal projection과 constrained assembly
- local→global feedback: active-region aero 재평가와 reduced predictor--corrector
- Gaussian 갱신: affine mean/covariance transport

## 14단계 solver map

1. 이전 상태 읽기: 저장된 global modal state, local residual state, gate state를 불러오는 state operation.
2. 현재 표면 복원: modal/local lifting과 deformation-gradient 기반 normal/area update. 네트워크 없음.
3. 상대풍과 surface force: wind-grid interpolation, relative velocity, quasi-steady drag/lift/friction law. full CFD/Navier--Stokes를 runtime에 풀지 않음.
4. object force 축약: offline에 선택한 aerodynamic samples와 weights로 empirical cubature; Galerkin projection으로 modal generalized force 생성.
5. global predictor: reduced mass/stiffness/damping system을 implicit Newmark 또는 backward Euler로 적분. structural restoring/damping은 항상 활성.
6. patch 오차 추정: 작은 topology-aware gate network가 global-only future error 또는 uncertainty를 출력. teacher global-projection error로 감독.
7. local 누락 힘: active patch GNN/MLP가 global state, patch geometry/history, local wind, material/boundary condition에서 missing local generalized force를 예측.
8. local 상태 적분: patch별 local reduced mass/stiffness/damping system에 predicted missing force를 넣어 implicit integration. 네트워크가 위치를 직접 출력하지 않음.
9. global 중복 제거: mass-orthogonal complement projector를 적용하는 고정 matrix operation.
10. patch 조립: displacement/velocity는 partition-of-unity로 연속 결합하고, force는 작은 constrained least-squares/KKT solve로 resultant force·torque와 internal action--reaction을 보존.
11. global feedback: active patch의 corrected geometry에서 normal/area/relative wind와 aero를 다시 계산하고, aero delta와 local reaction을 modal force로 투영해 작은 global corrector solve 수행.
12. 최종 anchor state: corrected global modal state와 local complement state를 deterministic lifting하여 anchor position/velocity/deformation gradient 생성.
13. Gaussian transport: anchor deformation gradient로 Gaussian mean, covariance, orientation/view frame을 affine/co-rotational update. 네트워크 없음.
14. 다음 상태 저장: global/local state와 hysteresis를 저장. inactive local network는 skip하되 남은 local energy는 cheap damped integration 후 종료.

## 학습 단계

고해상도 thin-shell FEM/MPM trajectory를 global basis에 투영한다. global-only reconstruction이 설명하지 못한 patch별 미래 오차를 gate target으로, teacher force/acceleration에서 global contribution을 제거한 값을 missing-force target으로 쓴다. CFD/LBM teacher가 있으면 wake와 unsteady aerodynamic effect가 missing-force label에 포함될 수 있다.

Runtime wind field는 prescribed input이다. Corrected geometry가 다음 aerodynamic load를 바꾸지만 object가 wind field 자체를 변화시키는 full two-way Navier--Stokes FSI는 아니다. 이를 포함하려면 별도 fluid solver가 필요하다.

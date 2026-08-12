# 2026-08-13 02 global/local 각 레벨 방법의 신규성 재검토

## 질문 정정

이번 검토의 대상은 `global--local hierarchy` 자체가 아니다. Wind-driven deformable static 3DGS라는 목표 도메인에서 다음 각 레벨의 구체적인 입력, 상태, 출력, 결합 연산이 이미 동일하거나 유사하게 사용되었는지를 판정한다.

- global: 현재 변형된 Gaussian thin surface에서 공기력을 계산하고, object-wide reduced elastic state로 투영·적분하는 방법
- local: global basis가 놓치는 flutter/fold를 오차가 큰 patch에서만 계산하고, basis complement와 보존 제약을 거쳐 다음 global dynamics에 되먹임하는 방법

## 제안하는 global level

구조 상태는 object-wide modal coordinate로 둔다.

```text
x(q) = x0 + Phi q
relative wind = W(x(q), t) - v(qdot)
current surface force = aero(current normal, current area, relative wind)
Q_aero = Phi^T f_aero
M_r qddot + C_r qdot + K_r q + g_nl(q) = Q_aero + Delta Q
```

실시간화를 위해 모든 Gaussian을 평가하지 않고, force/moment를 보존하도록 선택한 surface samples에서 generalized force를 근사하는 hyper-reduction을 고려한다.

### global level의 선행성과 남는 경계

- [Wind Projection Basis, CGF 2009](https://doi.org/10.1111/j.1467-8659.2009.01393.x)는 wind load를 global modes에 투영하고 modal oscillator를 적분한다. 따라서 `wind -> Phi^T f -> reduced second-order ODE`는 신규가 아니다.
- [Gaussian Swaying, WACV 2026](https://arxiv.org/html/2512.01306)은 현재 Gaussian의 normal, area, relative flow로 drag/friction/lift를 계산한다. 따라서 current-deformed Gaussian surface aero도 신규가 아니다.
- [DynamicTree, CVPR 2026](https://arxiv.org/html/2510.22213)은 static 3DGS tree에서 modal-like spectrum을 만들고 외력에 대한 reduced dynamics를 적분한다.
- [FreeForm, CVPR 2026](https://arxiv.org/html/2605.29318)과 [PhysSkin, CVPR 2026](https://arxiv.org/html/2603.23194)은 static geometry/3DGS의 mesh-free reduced deformation과 lifting에 가깝다.
- [DiffWind, ICLR 2026](https://zju3dv.github.io/DiffWind/)는 spatial wind와 3DGS-derived particles/MPM을 결합한다.

단일 선행에서 그대로 확인되지 않은 global 조합은 다음이다.

```text
independently reconstructed unordered static 3DGS
-> training-only mesh로 oriented thin-surface topology, attachment, bending/stretching operator distillation
-> target runtime mesh 없이 object-wide ROM
-> current-deformed normal/area, local relative velocity, spatial wind를 반영한 state-dependent generalized aero force
-> force/moment-preserving hyper-reduction
```

하지만 단순히 Gaussian Swaying의 공기력과 FreeForm/DynamicTree의 ROM을 연결하면 obvious composition으로 보일 위험이 높다. 신규성은 단순 연결이 아니라 unordered Gaussians에서 thin-shell operator를 복원하는 방법, 그리고 state-dependent spatial aero projection을 보존적으로 hyper-reduce하는 새 연산에 있어야 한다.

## 제안하는 local level

```text
global reconstruction: x_g = x0 + Phi q
error/uncertainty estimator -> active patch set A_t
local residual: delta x = P_perp sum_{p in A_t} B_p r_p
P_perp = I - Phi(Phi^T M Phi)^-1 Phi^T M
corrected state: x = x_g + delta x
re-evaluate aero/internal force on corrected state
project the result back to the next global step
```

inactive patch는 실제 network evaluation을 생략하고, overlapping patch의 residual은 total force와 torque를 보존하도록 조립한다.

### local level의 선행성과 남는 경계

- [Subspace Condensation, SIGGRAPH 2015](https://www.tkim.graphics/CONDENSE/)은 oracle이 필요한 local full-space 영역만 활성화하고 global subspace와 양방향으로 결합한다.
- [Trading Spaces, SIGGRAPH Asia 2024](https://www.dgp.toronto.edu/projects/trading-spaces/)는 residual/error와 progress를 평가해 영역별 local enrichment를 선택하고 modal--local coupling을 푼다.
- [Learning Contact Corrections, SIGGRAPH 2021](https://mslab.es/projects/LearningContactCorrections/)은 global reduced deformation 위에 learned high-frequency local correction을 더하고 correction의 Jacobian을 통해 force에 영향을 준다.
- [PGRD, 2026](https://arxiv.org/html/2607.13451)은 paper/flag를 포함한 physics backbone에 learned per-particle velocity residual을 더하고 이를 다음 rollout에 되먹임하며 3DGS로 렌더링한다.

따라서 error-triggered activation, learned local correction, global/local feedback은 각각 신규가 아니다. 조사 범위에서 그대로 확인되지 않은 것은 다음의 wind-driven 3DGS-specific local operator 조합이다.

```text
target-mesh-free inferred thin-shell patches
+ calibrated base-error/uncertainty activation and real compute skipping
+ mass-orthogonal modal-complement correction
+ overlap에서 resultant force/impulse와 torque 보존
+ corrected current geometry에서 aerodynamic force를 다시 계산
+ 그 generalized load가 다음 object-level state를 변경하는 two-way aeroelastic feedback
```

## 판정

- 각 기본 수식의 novelty: 낮음
- global level을 이루는 개별 연산의 novelty: 낮음에서 중간
- local level의 일반 수치 골격 novelty: 낮음
- 목표 도메인에 특화된 exact operator/coupling의 novelty: 중간, 구현 내용에 따라 중간 이상 가능

핵심은 `두 레벨을 썼다`가 아니라 다음 세 방법을 새롭게 풀어냈다는 것을 보여주는 것이다.

1. topology-distilled target-mesh-free Gaussian thin-shell ROM
2. current-state spatial aerodynamic load의 force/moment-preserving hyper-reduction
3. error-triggered complement-space local flutter correction과 conservative aeroelastic feedback

논문에서는 새 modal equation이나 새 drag law를 주장하지 않는다. 위 세 operator의 설계, 안정성, 보존성, target-mesh-free 특성, 그리고 quality--latency Pareto를 기여로 주장한다.

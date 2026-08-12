# 2026-08-12 01 adaptive RSH 논문 주제 검토

## 배경

사용자가 `ideas/3dgs_adaptive_rsh_residual_wind_response_2026-08-12_v3_bundle.zip`으로 논문 주제를 변경하고, 해당 방향의 연구 가치와 위험을 분석해 달라고 요청했다.

이번 검토는 번들 내부의 `.tex`, `.pdf`, `.bib`, revision/build note, 기존 2026-08-05 method redirection 기록, 그리고 2026-08-12 기준 공개 선행연구를 함께 대조했다. 공통 Startup Protocol에 적힌 다음 파일은 현재 workspace와 `/home/choi/projects/2026_paper_work` 아래에서 찾지 못했다.

- `../RESEARCH_PROJECT_GUIDE.md`
- `../templates/research_project/TEMPLATE_MANIFEST.md`

따라서 root/ideas `AGENTS.md`, 현재 ideas 기록, 최근 code/experiments/session 기록을 기준으로 검토했다.

## 번들 확인

- ZIP SHA-256: `4716e2e88b7deb2076c43a2bc53661155dca44d03681d930e22d9a0d25f09f2e`
- `unzip -t`: 모든 entry 통과
- 제공 PDF: 82 pages
- 내부 제목: `Adaptive Response-SH and Neural-Residual Wind Animation for Static 3D Gaussian Thin-Surface Assets`
- 번들의 핵심 전환:
  - source mesh는 training-only privileged teacher로 사용한다.
  - target runtime에는 static 3DGS, wind, material, attachment만 둔다.
  - patch별 canonical aerodynamic response를 vector real SH인 RSH로 예측·cache한다.
  - intrinsic deformation이 큰 patch에만 aerodynamic/structural neural residual을 실행한다.
  - topology-distilled anchors, global graph, second-order integration, affine MLS Gaussian transport를 결합한다.

## 종합 판정

현재 판정은 `조건부 진행(yellow / go-conditional)`이다.

| 항목 | 현재 평가 | 이유 |
|---|---|---|
| 문제 중요성 | 높음 | static 3DGS asset을 controllable wind animation으로 바꾸는 문제는 CG 응용 가치가 분명하다. |
| 시스템 이야기 | 높음 | target-mesh-free, no runtime continuum simulation, direct 3DGS rendering이라는 사용자 가치가 선명하다. |
| 현재 신규성 | 중간 이하 | aerodynamic SH, precomputed wind response, neural Gaussian dynamics, residual correction 각각에 가까운 선행연구가 있다. |
| 수식 일관성 | 수정 필요 | gate가 structural restoring까지 끄고, hysteresis activation이 불연속이며, target decomposition과 보존식에 충돌이 있다. |
| 전체 구현 가능성 | 낮음 | 19 modules, 4 main contributions, 새 teacher dataset, topology, dynamics, transport, appearance를 한 번에 요구한다. |
| 축소 버전 가능성 | 중간~높음 | attached single-layer sheet, oracle/simple topology, always-on linear structure, RSH/analytic probe, adaptive residual로 줄이면 검증 가능하다. |

아이디어는 2026-08-05의 plain local/global direct neural surrogate보다 CG representation으로서 더 명시적이고 falsifiable하다. 다만 현 상태의 `RSH + gated residual` 수식은 그대로 구현하면 핵심 claim을 뒷받침하지 못한다.

## 강점

1. RSH와 appearance SH를 `RSH`/`ASH`로 분리하여 의미 혼동을 줄였다.
2. RSH coefficient를 frame-to-frame 갱신하지 않고 co-rotated wind query를 사용하므로 coefficient drift와 pure rigid motion 처리가 명확하다.
3. direct-response network를 필수 baseline과 fallback으로 남기고, active ratio 기반 break-even을 실제 profiling으로 판단하도록 했다.
4. conservative area/mass, non-canceling Gaussian reliability, rank-2 shell map, affine MLS, pin/reaction projection 등 Revision 2의 중요한 구현 계약을 유지했다.
5. full tree, paper-airplane flight, full two-way FSI를 제외하고 attached/compliant-attached thin surface로 범위를 줄였다.
6. independent GS reconstruction variant, object-disjoint split, oracle topology upper bound, rigid/affine transport unit test 등 falsification-first 구조가 좋다.

## 신규성 경계

가장 중요한 누락 선행은 [OmniAD, SIGGRAPH 2015](https://studios.disneyresearch.com/2015/07/27/omniad-data-driven-omni-directional-aerodynamics/)다. OmniAD는 이미 임의 입사 풍향에 대한 공기력과 토크 계수를 spherical harmonics로 표현하여 rigid lightweight object를 실시간 시뮬레이션한다. 따라서 다음은 독립 신규 기여로 주장할 수 없다.

- 풍향별 aerodynamic force/torque를 SH에 저장하는 것
- SH coefficient를 미리 계산하여 runtime query를 싸게 만드는 것
- co-rotated local frame에서 directional response를 평가하는 큰 원리

[Wind Projection Basis, CGF 2009](https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1467-8659.2009.01393.x)도 tree의 wind load response를 precompute하여 임의 directional wind에 실시간 대응한다. 따라서 precomputed directional wind response 자체도 오래된 CG 계보다.

2026년 직접 경쟁선은 다음과 같다.

- [Gaussian Swaying, WACV 2026](https://openaccess.thecvf.com/content/WACV2026/papers/Yan_Gaussian_Swaying_Surface-Based_Framework_for_Aerodynamic_Simulation_with_3D_Gaussians_WACV_2026_paper.pdf): Gaussian surface patch normal/area에 직접 aerodynamic force를 적용하고 MPM과 shading을 결합한다.
- [DiffWind, ICLR 2026](https://zju3dv.github.io/DiffWind/): 3DGS-derived particles, wind field, MPM/LBM으로 wind reconstruction과 forward simulation을 수행한다.
- [Neural Gaussian Force Field, ICLR 2026](https://neuralgaussianforcefield.github.io/): 3D Gaussian representation에서 global/local neural force field와 ODE integration으로 interactive dynamics를 예측한다.
- [GausSim, ICCV 2025](https://www.mmlab-ntu.com/project/gausim/index.html): hierarchical Gaussian neural simulator와 mass/momentum constraints를 사용한다.
- [DynamicTree, CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/Li_DynamicTree_Interactive_Real_Tree_Animation_via_Sparse_Voxel_Spectrum_CVPR_2026_paper.html): static 3DGS tree에 sparse spectral motion representation과 외력 반응을 제공한다.
- [PhysSkin, CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/Lei_PhysSkin_Real-Time_and_Generalizable_Physics-Based_Animation_via_Self-Supervised_Neural_Skinning_CVPR_2026_paper.html): mesh-free, discretization-agnostic, real-time physics-based animation을 Gaussian Splatting을 포함한 여러 표현에 적용한다.
- [ParticleGS, CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/Quan_ParticleGS_Learning_Neural_Gaussian_Particle_Dynamics_from_Videos_for_Prior-free_CVPR_2026_paper.html), [Learning a Particle Dynamics Model with Real-world Videos](https://arxiv.org/abs/2605.23845): learned Gaussian/particle dynamics 자체의 novelty 경계를 만든다.
- [PGRD](https://arxiv.org/abs/2607.13451), [PhysCoRe](https://arxiv.org/abs/2607.20653), [DeformMaster](https://arxiv.org/abs/2605.09586): physics base와 neural residual의 결합 자체는 이미 혼잡한 영역이다.

따라서 방어 가능한 novelty는 `SH`나 `residual` 단독이 아니라 다음 결합이다.

```text
independently reconstructed static 3DGS
+ training-only mesh-privileged topology/mechanics supervision
+ target-mesh-free topology-distilled thin-surface patches
+ cached patch-local aerodynamic base
+ error/deformation-triggered nonlinear correction
+ no runtime continuum solver
+ affine Gaussian transport and direct rendering
```

## 구현 전 P0 수정 사항

### 1. Low-deformation structural dead zone

현재 RSH는 canonical geometry의 instantaneous aerodynamic load만 나타낸다. 그러나 최종 식은 structural restoring/damping까지 deformation gate 안에 넣는다.

```text
a_local = a_RSH + gamma * (delta_a_aero + delta_a_struct)
```

이 식에서는 inactive patch가 aerodynamic force만 받고 작은 strain/bending에 대한 탄성 복원력이 없다. 무풍으로 바뀌어도 deformation이 threshold 아래이면 잔류 변형이 남을 수 있고, wind load가 누적되다가 threshold에서 restoring force가 갑자기 켜질 수 있다.

권장 구조는 다음과 같다.

```text
a_local = a_RSH
        + a_struct_linear_always_on
        + gamma_aero * delta_a_aero_nonlinear
        + gamma_struct * delta_a_struct_nonlinear
```

cheap linear/corotational elastic and damping base는 항상 켜고, gate는 base가 설명하지 못하는 nonlinear correction만 제어해야 한다.

### 2. Smooth gate와 hysteresis skip의 불연속

현재 `gamma`는 `tau_off < d < tau_on`에서 0에서 1로 증가하지만, inactive patch는 `d >= tau_on`까지 residual network를 실행하지 않는다. 따라서 activation 순간 residual이 거의 full scale로 0에서 갑자기 나타난다. `continuous gamma avoids a jump`라는 설명과 모순이다.

다음 중 하나가 필요하다.

- `gamma > 0`인 transition band에서는 residual을 미리 계산하고 blend한다.
- compute-on threshold, full-blend threshold, compute-off threshold를 분리한다.
- activation 시 cached/zero residual에서 수 frame cross-fade한다.

### 3. RSH necessity가 아직 증명되지 않음

canonical RSH teacher target은 문서의 analytic drag/friction/lift 식을 frozen geometry에서 평가해 만든다. 이 식은 dot product와 scalar 연산 몇 번이므로, low-order SH basis 16--25개 평가보다 직접 analytic force가 더 싸고 정확할 가능성이 크다.

RSH가 유효하려면 다음 중 적어도 하나를 실제로 압축해야 한다.

- nonlocal shielding/exposure
- unresolved microgeometry
- patch-integrated complex aerodynamic response
- expensive CFD/measurement teacher response

그렇지 않으면 RSH는 analytic formula의 불필요한 근사층이 된다.

### 4. Target decomposition과 force 중복

현재 total teacher acceleration을 graph low/high-pass로 나눈 뒤 runtime에서 RSH, gravity, attachment projection을 다시 더하면 gravity, aerodynamic low-frequency component, constraint reaction이 이중 계산될 수 있다.

권장 순서는 다음이다.

```text
teacher residual
= total unconstrained teacher acceleration
  - gravity
  - RSH/analytic aerodynamic base
  - always-on structural base
  - explicitly handled attachment term
```

그 후에만 residual을 global/local로 분해해야 한다. Hard constraint reaction은 projection과 학습 target에서 중복 사용하지 않고, soft attachment는 force 또는 projection 중 하나의 convention으로 고정해야 한다.

### 5. 최종 patch blend 이후 보존이 깨짐

한 patch 안의 antisymmetric edge force는 linear momentum을 보존하지만, endpoint별 partition weight가 다르면 overlapping-patch blend 뒤 action--reaction이 취소되지 않는다. 또한 arbitrary vector pair force는 linear momentum만 보존하고 angular momentum은 보장하지 않는다.

- global material edge별 force를 한 번만 assemble하거나 symmetric edge blend weight를 사용한다.
- net force, net torque, angular momentum, reaction wrench를 최종 assembled field에서 검증한다.

### 6. Static 3DGS의 metric scale 부재

일반 SfM/3DGS는 절대 scale이 없다. 그런데 문서는 SI-like wind speed, area, mass, thickness, Young's modulus를 사용한다. User-supplied metric scale을 runtime contract에 넣거나, material/wind control을 명시적으로 nondimensional preset으로 제한해야 한다.

### 7. Gate input과 실제 approximation error가 불일치

RSH error는 deformation뿐 아니라 wind direction, SH fit confidence, topology error, exposure change에서도 커질 수 있다. 문서의 severity 식은 주로 deformation만 사용한다. 따라서 `deformation gate`보다 `base-model error/uncertainty gate`가 더 정확한 framing이다.

### 8. Canonical tangent gauge 불안정

원형·정사각형·등방성 patch에서는 PCA tangent axis가 GS reconstruction마다 임의로 회전하거나 sign flip할 수 있다. RSH coefficient predictor가 local-frame target을 학습하려면 안정된 tangent gauge, rotation-equivariant encoder, 또는 coefficient target의 in-plane equivariant 처리 중 하나가 필요하다.

## 제목과 framing 평가

현재 내부 제목은 component-driven이고 reviewer에게 다음 오해를 줄 수 있다.

- `Adaptive Response-SH`: 실제로 RSH coefficient는 immutable하고 adaptive한 것은 residual execution이다.
- `Neural-Residual`: 2026년 기준 너무 일반적이며 residual-dynamics 선행과 바로 충돌한다.
- `Response-SH`: 3DGS appearance SH와 제목에서 혼동될 수 있다.
- 가장 강한 차별점인 training-only mesh privilege, target-mesh-free topology distillation, no runtime continuum simulation이 제목에 보이지 않는다.

가장 안전한 주제명은 다음이다.

> **Topology-Distilled Adaptive Wind Animation of Static 3D Gaussian Thin Surfaces**

RSH를 제목에 반드시 남기려면 다음이 더 정확하다.

> **Deformation-Gated Aerodynamic Harmonic Caching for Wind Animation of Static 3D Gaussian Thin Surfaces**

다만 gate를 error-aware 방식으로 고치면 `Error-Gated Aerodynamic Response Caching`이 더 정확하다.

권장 연구 질문은 다음이다.

> Can a topology-distilled static 3D Gaussian thin-surface asset combine a cached aerodynamic base, an always-on lightweight structural response, and selectively executed nonlinear corrections to achieve a better quality--latency Pareto frontier than direct analytic, sparse-solver, and fully neural baselines without a target mesh or runtime continuum simulation?

## 한 편으로 줄이는 권장 범위

첫 논문에 유지:

- attached, single-connected thin surfaces
- flag, pennant, hanging cloth, strip 중심
- oracle 또는 simple normal-aware topology부터 시작
- analytic versus RSH aerodynamic base comparison
- always-on lightweight structural base
- always-on residual versus adaptive residual
- affine MLS mean/covariance transport
- synthetic quantitative evaluation와 소수 real static-GS demo

후속으로 이동:

- general topology distillation과 close disconnected layers
- detached SE(3) flight
- full tree, flower system
- opacity-based wind exposure
- appearance residual/relighting
- dynamic self-contact/contact graph
- arbitrary cross-family material generalization

Topology distillation을 첫 논문의 main contribution으로 반드시 유지한다면 adaptive RSH를 supporting efficiency module로 내리는 편이 낫다. 반대로 adaptive response representation을 main contribution으로 삼는다면 oracle/simple topology를 사용하여 두 난제를 동시에 풀지 않는 편이 안전하다.

## Falsification-first 실험 순서

### Gate A: Aerodynamic representation necessity

한 oracle patch에서 다음을 held-out wind direction error와 실제 GPU latency로 비교한다.

- direct analytic aerodynamic force
- directional LUT + interpolation
- small directional MLP
- fitted RSH upper bound
- predicted RSH

RSH가 quality--latency Pareto에 없으면 RSH를 headline에서 제거한다.

### Gate B: Small-deformation recovery and continuity

한 장의 flag에서 threshold crossing, weak-wind steady state, wind-stop recovery, activation jerk를 측정한다. Always-on structural base 없이 통과하지 못하면 현재 gate 식을 폐기한다.

### Gate C: Sparse solver fairness

동일한 anchors와 graph에서 mass-spring, XPBD 또는 projective dynamics를 실행한다. Neural method의 speedup이 단순히 MPM보다 simulation resolution이 낮아서 생긴 것인지 분리한다.

### Gate D: Direct learnability

Oracle topology의 한 asset에서 full direct network가 먼저 2--4초 teacher rollout을 안정적으로 학습해야 한다. 실패하면 RSH/residual/topology를 확장하지 않는다.

### Gate E: Adaptive Pareto

`direct network -> RSH/analytic base + always-on residual -> adaptive residual` 순서로 비교한다. 권장 초기 통과 기준은 다음이다.

- 4초 rollout normalized position error `< 5% L`
- adaptive가 always-on residual 대비 error 증가 `<= 5%`
- direct network 대비 dynamics latency `>= 2x` 개선
- end-to-end latency `>= 1.25--1.5x` 개선
- divergence 0, invalid covariance 0
- activation/deactivation jerk가 perceptually/quantitatively 제한됨

### Gate F: Topology generalization

Core dynamics가 성립한 후에만 predicted topology와 independent GS reconstructions를 추가한다.

- topology F1 `>= 0.90`
- hard-negative FPR `< 5%`
- inferred topology dynamics가 oracle error의 `10--15%` 이내

## 결론

`공력 SH + neural residual`이라는 부품 중심 주제는 선행연구와의 충돌 때문에 약하다. 반면 `mesh-privileged topology distillation을 통해 static 3DGS thin surface에 target-mesh-free wind animation을 제공하고, cheap base와 선택적 nonlinear correction으로 quality--latency Pareto를 개선한다`는 시스템 문제는 충분히 경쟁력이 있다.

따라서 v3를 바로 authoritative method로 채택하지 않는다. 먼저 P0 수식 수정과 Gate A--C를 수행하고, RSH가 analytic force 및 sparse solver보다 실제로 유리한지 확인한 뒤 paper headline을 확정한다.

## 변경 파일

- 추가: `ideas/sessions/2026-08-12_01_adaptive_rsh_topic_review.md`
- 기존 `idea_sketch.tex`, `idea_sketch.pdf`, `changelog.md`, bundle은 수정하지 않았다.

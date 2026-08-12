# 2026-08-13 01 multi-level dynamics 선행연구 검토

## 질문

Object 전체의 저차원/global dynamics로 큰 움직임을 계산하고, local flutter·wrinkle 등 설명하기 어려운 부분만 별도 correction으로 계산하는 multi-level 구조가 기존에 소개되었는지 검토했다.

## 결론

`global reduced dynamics + local high-frequency/selective correction`이라는 일반 원리는 이미 강한 선행이 있다. 다음 요소들도 각각 기존 연구에 존재한다.

- global modal/subspace simulation
- global basis에 local deformation basis를 runtime 추가
- 사건 주변만 full-space computation으로 활성화
- coarse cloth simulation에 learned detail을 추가
- low-frequency subspace와 localized high-frequency solver의 분리
- 3DGS에서 coarse-to-fine hierarchy
- 3DGS에서 global force와 local stress branch의 분리
- physics backbone과 learned residual의 결합

따라서 `multi-level`, `global-to-local`, `global modal + local residual` 자체를 main novelty로 주장하면 안 된다.

## 가장 가까운 classic 선행

1. [Subspace Integration with Local Deformations, SIGGRAPH 2013](https://cs.nyu.edu/~dzorin/papers/harmon2013sil.pdf)
   - smooth global deformation은 작은 modal basis로 계산한다.
   - load/contact 주변의 localized deformation은 runtime local basis로 보충한다.
   - active loaded region이 작으면 필요한 local function 수도 작다는 efficiency 논리를 사용한다.

2. [Subspace Condensation, SIGGRAPH 2015](https://www.tkim.graphics/CONDENSE/)
   - 평상시 물체 대부분은 global subspace에서 계산한다.
   - collision 등 basis 밖의 새로운 사건 주변만 full-space tetrahedra를 활성화한다.
   - reduced/full 영역을 양방향 결합하고 active 영역 크기에 따라 계산량이 달라진다.

3. [Physics-Inspired Upsampling for Cloth Simulation in Games, SIGGRAPH 2011](https://cal.cs.umbc.edu/Papers/Kavan-2011-PUF/index.html)
   - coarse cloth simulation을 learned upsampling으로 상세화한다.
   - wind-driven flag를 포함하며 coarse state로 설명되지 않는 high-frequency detail을 oscillatory modes로 추가한다.

4. [Subspace Clothing Simulation Using Adaptive Bases, SIGGRAPH 2014](https://publications.ri.cmu.edu/subspace-clothing-simulation-using-adaptive-bases)
   - full-space training data에서 pose/state별 low-dimensional basis pool을 만들고 runtime에 적응적으로 선택한다.

5. [Learning Contact Corrections for Handle-Based Subspace Dynamics, SIGGRAPH 2021](https://mslab.es/projects/LearningContactCorrections/)
   - handle-based global reduced deformation 위에 subspace가 놓친 nonlinear local contact correction을 학습해 추가한다.

6. [Subspace-Preconditioned GPU Projective Dynamics with Contact, SIGGRAPH Asia 2023](https://wanghmin.github.io/publication/li-2023-spg/Li-2023-SPG.pdf) 및 [Efficient GPU Cloth Simulation with Non-distance Barriers and Subspace Reuse, TOG 2024](https://arxiv.org/html/2403.19272)
   - cloth solver에서 low-frequency residual은 subspace가 처리하고 localized high-frequency residual은 full-order GPU iteration이 처리한다.
   - 이는 neural representation이 아니라 수치해법 분할이지만 global low/high split의 직접 선행이다.

## 3DGS/Gaussian dynamics 선행

1. [GausSim, ICCV 2025](https://www.mmlab-ntu.com/project/gausim/index.html)
   - Gaussian/CMS hierarchy를 구성하고 coarse-to-fine으로 deformation gradient를 전파한다.
   - kernel-wise computation을 약 95% 줄였다고 보고한다.
   - 고정 hierarchy이며 error-triggered active patch나 modal complement residual은 아니다.

2. [NGFF, ICLR 2026](https://arxiv.org/html/2602.00148)
   - object graph의 global 6-DoF force와 point-wise local stress를 분리하고 second-order ODE로 적분한다.
   - global branch는 rigid motion이며 elastic global modes는 아니지만, `global/local Gaussian force decomposition`이라는 일반 claim은 선점한다.

3. [DynamicTree, CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/Li_DynamicTree_Interactive_Real_Tree_Animation_via_Sparse_Voxel_Spectrum_CVPR_2026_paper.html)
   - compact sparse voxel spectrum을 modal basis처럼 사용하여 static tree 3DGS의 외력 반응을 계산한다.
   - local selective residual은 없고 animated mesh에 Gaussians를 bind한다.

4. [PhysSkin, CVPR 2026](https://zju3dv.github.io/PhysSkin/)
   - continuous skinning basis로 reduced coordinates를 full-space deformation에 lift한다.
   - mesh-free/discretization-agnostic reduced animation과 3DGS 적용이 이미 가깝다.

5. [PGRD, 2026](https://arxiv.org/abs/2607.13451)
   - spring-mass physics backbone에 learned per-particle residual을 더한다.
   - paper/flag와 3DGS rendering을 포함하므로 physics+residual+Gaussian 조합의 직접 경쟁선이다.

## 방어 가능한 좁은 차별점

조사한 범위에서는 다음 전체 조합은 찾지 못했다.

```text
training-only privileged mesh
-> independently reconstructed static 3DGS용 thin-surface topology/mechanics distillation
-> target-mesh-free object-wide reduced aeroelastic state
-> base-model predicted error/uncertainty가 큰 patch만 실제 계산
-> global-basis complement에 제한된 local correction
-> local correction의 global state/force feedback
-> Gaussian mean/covariance affine transport
```

이는 구성 요소의 신규성보다 문제 설정, 결합, 물리 제약과 측정된 quality--latency Pareto의 신규성이다.

## 설계상 필요한 차별화

- local correction을 단순히 더하지 않고 global basis가 이미 표현한 운동과 중복되지 않게 complement로 제한한다.
- deformation magnitude가 아니라 base model의 실제 predicted error/uncertainty로 active patch를 고른다.
- inactive patch의 network evaluation을 실제로 생략해 wall-clock 이득을 증명한다.
- local detail이 시각적 후처리에 그치지 않고 object-level generalized force/state에 feedback되도록 한다.
- overlap halo, partition-of-unity, hysteresis/cross-fade와 final assembled force/wrench 검증을 사용한다.

## 논문 framing 권고

피해야 할 claim:

- first hierarchical Gaussian simulator
- first global/local Gaussian dynamics
- first global modal + local residual simulator
- first adaptive local/full-space activation
- first static-3DGS reduced physics
- first directional modal wind response

권장 framing:

> `Topology-distilled, target-mesh-free thin-surface wind dynamics for static 3DGS with error-triggered local correction`

Multi-level 구조는 핵심 신규성보다 efficiency mechanism으로 두고, main contribution은 target mesh 없는 independently reconstructed 3DGS에서 wind-specific thin-surface topology와 dynamics를 distill하는 과정으로 유지한다.

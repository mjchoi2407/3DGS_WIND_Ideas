# 2026-08-12 02 object-level deformation-aware response 설계 검토

## 질문

Patch 단위 RSH가 전체 object deformation과 장거리 구조 응답을 놓치는 문제를 해결하기 위해 다음 두 방향을 비교했다.

1. 현재 deformation을 고려하는 object-level RSH
2. RSH 없이 현재 state에서 object response를 직접 계산하거나 예측하는 방법

이번 기록은 방향 분석만 남긴다. 사용자가 연구 방향 변경을 확정하지 않았으므로 `idea_sketch.tex`와 `changelog.md`는 수정하지 않았다.

## 핵심 판정

- 단순한 dense object-level per-anchor RSH는 권장하지 않는다. Patch를 하나로 합칠 뿐 현재 deformation을 자동으로 반영하지 않으며, state-conditioned coefficient를 매 frame 생성하면 RSH cache의 이점도 사라진다.
- 가능한 전역 RSH는 object deformation을 저차원 modal state로 표현하고, 각 mode에 작용하는 generalized aerodynamic force만 SH로 근사하는 `modal/state-space RSH`다.
- 현재 v3처럼 RSH target이 canonical geometry의 단순 analytic drag/friction/lift라면, current deformed normal/area에서 analytic force를 직접 다시 계산하는 것이 더 정확하고 더 쌀 가능성이 높다.
- 현 단계의 권장 mainline은 `current-geometry analytic aero + always-on reduced structural dynamics + optional direct neural correction + selective local flutter residual`이다.
- RSH는 headline contribution이 아니라 expensive nonlocal aerodynamic teacher를 압축할 필요가 입증될 때의 optional angular factorization으로 두는 편이 안전하다.

## 문제의 올바른 상태 정의

`wind direction -> final deformation`은 일반적으로 잘 정의되지 않는다. 같은 풍향에서도 현재 displacement, velocity, 이전 gust와 flutter phase에 따라 다음 response가 달라진다. 따라서 최소 상태는 다음과 같아야 한다.

```text
state = current deformation q
      + modal velocity q_dot
      + material/attachment
      + current wind descriptor
      + 필요한 경우 aerodynamic memory h
```

최종 displacement를 정적으로 조회하는 대신 generalized force 또는 acceleration을 예측하고 dynamics를 적분해야 resonance, wind-stop recovery, phase가 보존된다.

## 권장 object-level reduced dynamics

Anchor displacement를 object deformation basis로 줄인다.

```text
x_t ~= x_0 + Phi q_t
```

여기서 `Phi`는 vibration mode, graph/spectral mode, POD 또는 learned deformation basis이며, `q_t`는 object 전체 변형을 나타내는 저차원 coordinate다. Dynamics는 다음처럼 구조 base를 항상 유지한다.

```text
M_r q_ddot + C_r q_dot + K_r q + g_nl(q)
    = Q_aero(q, q_dot, h, W) + Q_gravity + Q_attachment
```

Global sway, attachment tension, 저주파 bending은 `q`가 담당하고, 작은 basis로 설명되지 않는 local flutter와 wrinkle만 local residual/decoder가 보충한다.

## RSH를 유지하는 전역형

### 피해야 할 dense state-conditioned coefficient field

```text
f_aero(q, omega) = C(q) Y_L(omega)
```

`C(q)`가 모든 anchor의 `3K x B` coefficient를 매 frame 출력하면, 현재 frame에서 필요한 풍향은 하나인데 `B=(L+1)^2`개 방향 계수를 모두 생성한다. 이 방식은 direct `f(q, omega)`보다 출력량이 크고 immutable cache라는 기존 장점도 잃는다.

### 가능한 low-rank modal RSH

Full nodal force 대신 `M`개의 generalized force에 대해 고정된 저랭크 state-direction factorization을 사용한다.

```text
Q_aero(q, q_dot, omega)
  = q_inf [H_0 + sum_j alpha_j(q, q_dot, h) H_j] Y_L(omega)
```

여기서 `H_j`는 setup 시 cache하고, runtime network는 작은 수의 scalar modulation `alpha_j`만 출력한다. 구조식은 항상 적분하며 RSH는 외부 aerodynamic load에만 사용한다.

이 방식은 다음 조건에서만 설득력이 있다.

- object motion이 작은 수의 deformation modes에 머문다.
- wind가 uniform하거나 2--8개의 macro-zone으로 충분하다.
- expensive CFD, shielding 또는 unresolved microgeometry response를 압축한다.
- 동일 deformation state에서 여러 풍향을 반복 query한다.
- low SH order와 작은 state rank로 held-out direction error가 충분히 작다.

Spatial gust가 중요하면 zone별 wind descriptor를 합산하거나 sampled wind field encoder를 써야 한다. Zone 수가 patch 수에 가까워지면 object-level 압축 이점은 사라진다.

## RSH-free 후보

### 1. Current-deformed analytic aero

```text
Q_aero = Phi^T f_analytic(x_t, v_t, W_t)
```

현재 normal, area, relative velocity를 사용해 local aerodynamic force를 직접 계산하고 modal space로 projection한다. v3의 teacher가 같은 analytic law라면 근사 오차가 없고, anchor당 dot product와 소수 scalar/vector 연산만 필요하다. `L=3/4` RSH의 16/25 basis 평가와 coefficient contraction보다 쌀 가능성이 있다.

### 2. Direct modal force operator

```text
Q_aero = N_theta(q, q_dot, h, wind_descriptor, material, attachment)
```

현재 필요한 `M`개 generalized force만 출력한다. 한 frame에 한 wind query를 하는 animation runtime에서는 state-conditioned RSH보다 자연스럽다. SH의 smooth angular inductive bias만 필요하면 coefficient field를 만들지 않고 `Y_L(omega)`를 network input encoding으로 넣을 수 있다.

### 3. Full-object graph operator

Object graph 전체의 current position, velocity, wind, material과 pins를 입력하고 node acceleration 또는 antisymmetric edge force를 직접 예측한다. Spatial wind와 local fold에는 가장 유연하지만, sparse message passing 비용, rollout drift, 넓은 training state coverage와 기존 MeshGraphNets/NGFF 계열의 신규성 충돌이 크다.

### 4. 권장 hybrid

```text
analytic current-geometry aero
+ always-on modal/corotational structural base
+ direct modal nonlinear correction
+ selective local flutter residual
```

이 구조는 전역 장거리 응답, 안정적인 복원, 현재 deformation의 aerodynamic 변화와 local detail을 분리한다. Neural gate는 structural base를 끄지 않고, base approximation error가 큰 correction만 선택적으로 실행한다.

## 방법 선택 규칙

| 조건 | 우선 후보 |
|---|---|
| Teacher가 local quasi-steady drag/lift | current-deformed analytic aero |
| 매 frame 현재 풍향 하나만 query | direct modal force operator |
| CFD, shielding, microgeometry response가 비쌈 | modal RSH 또는 low-rank state-direction factorization |
| 동일 state에서 많은 풍향을 반복 query | state-conditioned modal RSH 가능 |
| Spatial gust, fold, exposure 변화가 큼 | sampled-wind direct graph/modal operator |
| Global sway가 중요하고 flutter가 제한적 | global modes + local residual |

## 필수 반증 실험

Oracle topology flag 한 장에서 동일 anchors, mass, pins, integrator와 always-on structural base를 사용한다. 비교 대상은 다음 네 가지다.

1. `canonical modal RSH`
2. `low-rank deformation-conditioned modal RSH`
3. `RSH-free direct generalized-force MLP`
4. `recurrent state-space/GRU generalized-force model`

별도의 강한 기준선으로 `current-deformed analytic aero + sparse corotational shell/XPBD/PD`를 둔다. 다음을 측정한다.

- held-out yaw/elevation/speed와 held-out deformation
- sinusoidal gust, frequency chirp, abrupt wind start/stop
- traveling spatial gust
- 4--10초 open-loop rollout
- generalized-force 및 net-wrench error
- mass-normalized position error
- tip amplitude, dominant frequency, phase와 damping envelope
- energy growth/divergence
- single object와 batch object의 p50/p95 GPU latency, kernel 수와 memory

RSH를 main method로 유지하려면 적어도 다음을 보여야 한다.

- direct object model 대비 rollout error 증가 10% 이하
- tip frequency error 5% 이하, amplitude error 10% 이하
- dynamics latency direct 대비 1.5배 이상 개선
- batch latency 1.3배 이상 개선
- analytic aero가 놓치는 expensive/nonlocal effect의 error를 동일 비용에서 의미 있게 감소

마지막 조건을 만족하지 못하면 RSH를 제거한다. Modal hybrid는 작은 rank에서 deformation energy 대부분을 설명하면서 direct graph model보다 유의미하게 빨라야 한다.

## 신규성 경계

- [Wind Projection Basis](https://diglib.eg.org/items/5125d70b-aecc-4b9d-bc88-a716125e96bb)는 이미 object/tree 전체의 modal wind response를 precompute한다.
- [OmniAD](https://studios.disneyresearch.com/2015/07/27/omniad-data-driven-omni-directional-aerodynamics/)는 rigid object 전체의 force/torque coefficient를 spherical harmonics로 표현한다.
- [MeshGraphNets](https://arxiv.org/abs/2010.03409)는 current state, velocity와 wind vector에서 flag dynamics를 RSH 없이 직접 예측한다.
- [Data-driven Unsteady Aeroelastic Modeling](https://arxiv.org/abs/2111.11299)은 deformation과 aerodynamic memory를 low-order state-space로 결합한다.
- DynamicTree, GausSim, NGFF가 각각 compact global Gaussian motion, hierarchical Gaussian simulation, global/local neural force-field의 직접 경쟁선을 만든다.

따라서 `object-wide response`, `modal wind response`, `object-level SH` 자체는 headline novelty로 약하다. 방어 가능한 중심 기여는 다음 결합이다.

```text
training-only mesh teacher
-> independently reconstructed static 3DGS용 object-wide reduced state/operator distillation
-> target mesh 없는 always-on stable dynamics
-> global deformation + selective local flutter
-> direct Gaussian transport/rendering
```

## 최종 권고

현 v3에서 patch RSH를 dense object RSH로 확대하지 않는다. 먼저 다음의 RSH-free 기준 모델을 만든다.

```text
current-deformed analytic aerodynamic load
+ object-level reduced structural state-space
+ optional direct modal nonlinear correction
+ local high-frequency residual
```

그 후 directional response가 실제로 low-order SH와 작은 state rank로 압축되고 end-to-end latency 이득이 측정될 때만 `modal RSH`를 efficiency ablation 또는 supporting module로 채택한다. 이 판단이 통과되기 전에는 논문 제목과 핵심 claim에 RSH를 넣지 않는 편이 안전하다.

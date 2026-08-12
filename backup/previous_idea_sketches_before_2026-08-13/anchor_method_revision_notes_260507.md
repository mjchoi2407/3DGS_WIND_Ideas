# Anchor Method Revision Notes

For: **Wind-Driven Deformable 3D Gaussian Splatting: Idea Sketch**  
Date: 2026-05-07

## Core decision

Use a **Fixed-topology Gaussian-aware material anchor mesh**.

The method should **not** perform per-frame remeshing or per-frame Gaussian rebinding. Adaptive remeshing/anchor placement happens once during preprocessing or asset setup. Runtime updates only anchor states, wind exposure, loads, local frames, and Gaussian attributes.

## Final one-line definition

> Use a fixed-topology Gaussian-aware material anchor mesh: anchors are persistent mesh vertices, placement is Gaussian/wind-aware, runtime updates states only, and Gaussians are transported through persistent triangle-local bindings.

## Expression candidates to preserve

### Primary / method-name candidates

- Fixed-topology Gaussian-aware material anchor mesh
- Gaussian-aware fixed-topology material anchor mesh
- Persistent Gaussian-Aware Material Anchors
- Gaussian-aware persistent simulation anchors
- Wind-aware Gaussian-to-proxy material anchoring
- Mesh-derived simulation anchors
- Physics anchors / mesh-proxy anchors
- Simulation node / physics node
- Adaptive Simulation Mesh Anchoring *(preprocessing-only; avoid runtime ambiguity)*
- 3DGS-derived adaptive simulation anchor mesh *(preprocessing-only)*

### Anchor type terms

- simulation anchors: `V_sim` vertices
- constraint anchors: pinned/support/user-selected vertex subset
- load anchors: face/vertex quadrature sites for wind load projection
- binding anchors: triangle-local binding references
- material-space anchors
- persistent material anchors
- time-varying anchor states

### Pipeline/runtime terms

- one-time simulation-oriented proxy construction
- fixed-topology mesh-proxy simulation
- persistent Gaussian-to-mesh binding
- triangle-local Gaussian binding
- mesh-local frame transport
- Gaussian attribute transport
- Gaussian-opacity-based wind exposure
- conservation-preserving dense-to-proxy force projection
- deformation-aware SH residual correction
- optional runtime LOD over fixed anchors
- active-anchor LOD
- constraint LOD

### Phrases to avoid

- adaptive remeshing during simulation -> use **one-time simulation-oriented remeshing**
- dynamic anchor adjustment -> use **time-varying anchor states**
- per-frame proxy reconstruction -> use **fixed-topology runtime simulation**
- time-varying Gaussian binding -> use **persistent Gaussian-to-mesh binding**
- direct Gaussian-patch aerodynamics -> use **mesh-proxy wind load projection**
- AI SH coefficient prediction -> use **deformation-aware SH residual correction**

## Anchor construction

```text
s(p) = w_G * rho_G(p)
     + w_E * |grad e_G(p)|
     + w_B * B(p)
     + w_K * |kappa(p)|
     + w_C * C_constraint(p)
     + w_M * M_material(p)
     + w_F * F_projection_risk(p)

h(p) = clamp(h0 / sqrt(1 + s(p)), h_min, h_max)
```

`A_sim = V_sim`, where `V_sim` is the vertex set of the fixed-topology simulation proxy.

## Runtime invariant

```text
Preprocessing invariant:
  V_sim, E_sim, F_sim, constraint graph, Gaussian binding = fixed

Runtime variables:
  x_i(t), v_i(t), e_i(t), f_i(t), R_T(t), mu_j(t), Sigma_j(t), c_j(t) = updated

Not performed at runtime:
  remesh, rebind, proxy reconstruction, topology change
```

## Gaussian transport

```text
Binding for Gaussian g_j:
  B_j = (T_j, b_j, R_Tj^0, delta_j)

Mean transport:
  mu_j(t) = sum_{k in T_j} b_jk * x_k(t) + R_Tj(t) * delta_j

Covariance transport:
  Sigma_j(t) = R_Tj(t) * Sigma_j^0 * R_Tj(t)^T

Appearance correction:
  c_j(t) = R_SH(Delta T_j(t)) * c_j^0 + Delta c_j(t)
```

## Paste-ready contribution statement

> We introduce a fixed-topology Gaussian-aware material anchor mesh that serves as a persistent simulation scaffold for 3D Gaussian assets. The anchor distribution is optimized for Gaussian coverage, wind exposure variation, boundary constraints, thin-structure preservation, and force projection stability, enabling force-space wind dynamics and stable Gaussian attribute transport.

## Paste-ready method paragraph

> Our method constructs the simulation proxy once in the canonical configuration. During runtime, the proxy topology, material anchors, constraint graph, and Gaussian-to-mesh bindings remain fixed; only anchor states, wind exposure, external loads, local frames, and Gaussian attributes are updated. This design avoids frame-wise remeshing and rebinding, which are expensive and can introduce temporal popping.

## Korean summary paragraph

> 본 연구는 매 프레임 remeshing을 수행하지 않는다. 입력 3DGS로부터 fixed-topology Gaussian-aware material anchor mesh를 초기 1회 구성하고, animation 중에는 이 mesh의 vertex state만 갱신한다. Gaussian은 proxy triangle에 지속적으로 binding되어 local frame deformation을 통해 mean, covariance, appearance parameter가 이동한다. Anchor 배치는 단순 geometry simplification이 아니라 Gaussian coverage, wind exposure 변화, boundary constraint, thin structure, material variation, force projection stability를 반영한다.

## Suggested revision prompt for VSCode/coding agent

```text
Read docs/anchor_method_revision_notes.pdf and update the current Wind-Driven Deformable 3D Gaussian Splatting idea sketch accordingly.

Revision goals:
1. Add the recommended method name: "Fixed-topology Gaussian-aware material anchor mesh".
2. Clarify that the method does NOT perform per-frame remeshing or per-frame Gaussian rebinding.
3. State that adaptive remeshing/anchor placement happens only once during preprocessing or asset setup.
4. Define simulation anchors as persistent material-space mesh vertices: A_sim = V_sim.
5. Add the anchor placement criteria: Gaussian coverage, wind exposure variation, boundary constraints, thin-structure preservation, material variation, and force projection stability.
6. Add the runtime invariant: topology, constraint graph, and Gaussian bindings are fixed; only anchor states, wind exposure, loads, local frames, and Gaussian attributes are updated.
7. Insert paste-ready contribution and method paragraphs from the PDF into the contribution and method sections.
8. Update the mathematical formulation with the anchor score field s(p), target edge length h(p), persistent binding B_j, and Gaussian transport equations.
9. Add ablations: uniform anchors vs. Gaussian-aware anchors, Gaussian-as-anchor vs. mesh material anchors, persistent binding vs. per-frame rebinding, and no exposure-aware placement.
10. Add a 2026-05-07 decision log entry describing this anchor-method update.

Keep the original paper framing: 3DGS input -> simulation proxy -> wind dynamics -> Gaussian attribute transport -> 3DGS rendering. Do not reframe the paper as an AI SH coefficient prediction paper.
```

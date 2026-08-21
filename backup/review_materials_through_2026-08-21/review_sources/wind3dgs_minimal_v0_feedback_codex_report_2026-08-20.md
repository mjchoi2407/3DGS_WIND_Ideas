---
title: "Minimal V0 피드백 보고서 및 Codex 잔여 패치 명세"
document_type: "codex_feedback_patch_spec"
date: "2026-08-20"
source_file: "붙여넣은 텍스트 (1).txt"
source_sha256: "b1d4afa0736ed8c4df019b844effd6dc7a2c74bbd5122334f0d9629f2e8c23e1"
source_lines: 1938
scope_policy: "No new solver/module unless Gate failure or observed visible artifact requires it"
---

# Minimal V0 피드백 보고서 및 Codex 잔여 패치 명세

## 0. 문서 역할

이 파일은 현재 Minimal V0 canonical source에 대한 피드백 보고서이자 VS Code Codex extension에 직접 입력할 수 있는 패치 명세다. Equation label을 1차 기준으로 사용하고 source line은 위 SHA-256 revision에서만 유효한 보조 기준으로 사용한다.

모든 실제 지적은 다음을 포함한다.

- 정확한 equation label 또는 source line
- 원문의 LaTeX 원문 전사
- 반례 또는 차원 분석
- 수용 / 반박 / 감수 / 후속 보류 판정
- Minimal V0 scope 영향
- 허용되는 replacement LaTeX와 acceptance test

### 절대 스코프 규칙

- 새 solver, shell backend, KKT/Schur, same-frame corrector, learned selector, hyper-reduction, 추가 runtime network 또는 병렬 mainline을 만들지 않는다.
- 이 보고서가 지정하지 않은 equation, threshold, config, asset class, file structure 또는 formatting을 정리하지 않는다.
- 기존 `corotated_tangent_plane_v2`, `generalized_eigen_v1`, `tangent_fit_v1`, Active/Decay policy와 single coupled solve를 변경하지 않는다.
- `KEEP / NO PATCH` 항목은 코드나 수식을 변경하지 않는다.
- source SHA-256이 다르면 자동 패치를 중단하고 equation label로 revision을 다시 확인한다.

## 1. 종합 판정

현재 source에는 새 solver/module을 요구하는 치명적 method 오류가 없다. 이전 수정 사항인 privileged-teacher 명칭, area-head 생성식과 dimensionless PSD는 정상 반영되었다.

다만 canonical implementation contract 관점에서 다음 네 항목은 닫아야 한다.

| ID | 항목 | Priority | 판정 | 적용 시점 |
|---|---|---|---|---|
| FB-01 | Anchor scalar mass에서 full mass matrix `M`을 만드는 규칙 누락 | `PATCH BEFORE V0-01` | 수용 | Global/Local basis build 이전 |
| FB-02 | `f_max` hard bound와 finite-force clamp의 정확한 의미 누락 | `PATCH BEFORE V0-02` | 수용 | full-anchor aero 구현 이전 |
| FB-03 | V0-R1 training-entry pre-check에서 `nu_decode` 누락 | `PATCH BEFORE V0-R1` | 수용 | predicted-scaffold training 이전 |
| FB-04 | privileged teacher의 per-unit counterfactual base state/rollout 규칙 누락 | `PATCH BEFORE V0-03 / GATE B` | 후속 보류 | teacher-ranked Local 실행 이전 |
| FB-05 | 이전 반영 사항 및 핵심 backend 회귀 확인 | `KEEP / NO PATCH` | 수용/감수 | 그대로 유지 |

### 구현 동결 판정

- V0 architecture/method freeze: **GO 유지**
- V0-00: 진행 가능
- V0-01 entry: `FB-01` 반영 전 차단
- V0-02 entry: `FB-02` 반영 전 차단
- V0-R1 training entry: `FB-03` 반영 전 차단
- V0-03/Gate B teacher run: `FB-04` 동결 전 차단

네 항목은 모두 기존 stage 내부의 수식 또는 manifest 계약 보완이다. 논문 novelty, solver 종류, runtime stage 수와 asset scope를 늘리지 않는다.

## 2. Codex 실행 순서

1. 현재 파일 SHA-256을 확인한다. 불일치하면 패치를 적용하지 않는다.
2. `FB-01`에서 `eq:minimal-area-mass`를 동일 label로 교체하고 lumped mass type을 package hash에 추가한다.
3. `FB-02`에서 `eq:minimal-aero`를 raw force + norm clamp로 동일 label 안에서 교체하고 force-clamp trace를 추가한다.
4. `FB-03`에서 lifecycle step 2의 field list에 `nu_decode`만 추가한다.
5. `FB-04`에서 teacher counterfactual common state와 per-unit score를 정의한다. Exact subset search는 구현하지 않는다.
6. XeLaTeX 2-pass, duplicate/missing label 검사와 아래 acceptance test를 수행한다.
7. 관련 없는 변경이 diff에 있으면 되돌린다.

# 3. 상세 피드백

## FB-01 - Full anchor mass matrix `M` 정의 누락

- **PRIORITY:** `PATCH BEFORE V0-01`
- **판정:** **수용**
- **Equation label:** `eq:minimal-area-mass; eq:minimal-global-basis; eq:minimal-coupled-blocks`
- **Source line:** `585--590; 682--695; 1083--1089`

### 원문의 LaTeX 전사

```latex
\begin{equation}
  a_i^0>0,\qquad
  \sum_i a_i^0 \approx A_{\mathrm{asset}},\qquad
  m_i=\rho_A a_i^0,\qquad
  \sum_i m_i \approx M_{\mathrm{asset}}.
  \label{eq:minimal-area-mass}
```

`M`은 이후 다음과 같이 소비된다.

```latex
V0 canonical Global basis는
\(\texttt{basis\_type=generalized\_eigen\_v1}\) 하나로 고정한다.
Free-coordinate fixed-Hessian package operator의 generalized eigenproblem에서 eigenvalue가 낮은
accepted \(r_g\)개 mode를 순서대로 사용한다 \cite{barbic2005stvk,diener2009windbasis}.
Hard-attachment free map을 \(N_f\)라 두면
\(M_f=N_f^\top MN_f\), \(K_f=N_f^\top KN_f\)이고,
full anchor basis는 \(\Phi=N_f\widetilde\Phi\)로 lift한다.

\begin{equation}
  K_{\mathrm f}\widetilde\Phi
  =
  M_{\mathrm f}\widetilde\Phi\Lambda,\qquad
  \Phi^\top M\Phi=I.
  \label{eq:minimal-global-basis}

\begin{equation}
\begin{aligned}
  M_{\mathcal S_t}&=B_{\mathcal S_t}^\top MB_{\mathcal S_t},\\
  C_{\mathcal S_t}&=B_{\mathcal S_t}^\top CB_{\mathcal S_t},\\
  K_{\mathcal S_t}&=B_{\mathcal S_t}^\top KB_{\mathcal S_t}.
\end{aligned}
  \label{eq:minimal-coupled-blocks}
```

### 반례 또는 차원 분석

현재 source는 scalar anchor mass

\[
 m_i=\rho_A a_i^0
\]

만 정의하고 full positional DOF의 mass matrix \(M\in\mathbb R^{3K\times3K}\)를 정의하지 않는다. 예를 들어 두 scalar DOF에서 동일한 diagonal mass를 가지는

\[
 M_A=
 \begin{bmatrix}1&0\\0&1\end{bmatrix},
 \qquad
 M_B=
 \begin{bmatrix}1&\gamma\\\gamma&1\end{bmatrix},
 \qquad 0<\gamma<1
\]

는 모두 symmetric positive definite이며 diagonal anchor mass도 동일하다. 그러나 같은 stiffness \(K\)에 대해

\[
 K\phi=\lambda M_A\phi
 \qquad\text{와}\qquad
 K\phi=\lambda M_B\phi
\]

의 generalized eigenvectors와 eigenvalues는 일반적으로 다르다. 따라서 다음이 구현체마다 달라질 수 있다.

\[
 \Phi,\quad \Psi_r,\quad M_{\mathcal S},\quad
 A_{\mathcal S}^{\mathrm{MP}},\quad
 \|\cdot\|_M,\quad \eta_{\mathcal G}.
\]

차원상

\[
 [m_i]=M,
 \qquad
 \left[\frac12\dot x^\top M\dot x\right]
 =ML^2/T^2
\]

이어야 하므로 \([M]=M\)다. \(\Phi^\top M\Phi=I\)까지 포함하면 basis column의 단위는 \(M^{-1/2}\)이며, 이는 현재 release proxy의 \(M\)-norm과 일치한다.

### 판정

**수용.** Canonical source가 한 구현을 소유하려면 lumped anchor mass를 명시해야 한다. Consistent mass나 learned mass를 추가할 이유는 없다.

### 허용되는 최소 replacement LaTeX

`eq:minimal-area-mass` 전체를 동일 label로 다음과 같이 교체한다.

```latex
\begin{equation}
\begin{aligned}
  a_i^0&>0,\qquad
  \sum_i a_i^0 \approx A_{\mathrm{asset}},\\
  m_i&=\rho_A a_i^0,\qquad
  \sum_i m_i \approx M_{\mathrm{asset}},\\
  M&=\operatorname{blkdiag}_{i=1}^{K}(m_i I_3)
  \in\mathbb R^{3K\times3K}.
\end{aligned}
  \label{eq:minimal-area-mass}
\end{equation}
```

바로 뒤 문단에 다음 구현 계약을 추가한다.

```latex
Canonical mass-matrix type은
\(\texttt{mass\_matrix\_type=lumped\_anchor\_v1}\)로 고정한다.
\(m_i\), full \(M\), free map \(N_f\)와
\(M_f=N_f^\top MN_f\)를 structural package hash에 저장한다.
```

### Codex action

- Dense matrix가 필요하지 않은 구현에서는 `mass_diag = repeat(m_i, 3)`로 저장해도 된다.
- 수학적 의미는 반드시 `blkdiag(m_i I_3)`와 동일해야 한다.
- Hard attachment는 기존처럼 `M_f=N_f^T M N_f`로 처리한다.
- Off-diagonal mass coupling, learned mass, mesh-consistent mass를 추가하지 않는다.

### Acceptance test

- [ ] 각 anchor의 3x3 block이 정확히 `m_i I_3`이다.
- [ ] 서로 다른 anchor block 사이의 off-diagonal entry가 0이다.
- [ ] 모든 `a_i^0>0`, `rho_A>0`에서 `M`과 `M_f`가 SPD다.
- [ ] `sum_i m_i`가 package convention의 `M_asset`와 tolerance 내 일치한다.
- [ ] SI/nd parity에서 generalized eigenvalue와 reconstructed trajectory가 허용오차 내 일치한다.
- [ ] `mass_matrix_type`, `m_i`, `M` 또는 equivalent diagonal array가 package hash에 포함된다.

### Minimal V0 scope 영향

- matrix/diagonal assembly 한 건
- package field/hash 한 건
- solver, basis algorithm, state policy와 runtime stage 변화 없음

---

## FB-02 - `f_max`와 finite-force clamp semantics 누락

- **PRIORITY:** `PATCH BEFORE V0-02`
- **판정:** **수용**
- **Equation label:** `eq:minimal-operating-envelope; eq:minimal-aero`
- **Source line:** `257--271; 901--914`

### 원문의 LaTeX 전사

```latex
Canonical run의 operating envelope는
\begin{equation}
  \mathcal E_{\mathrm{op}}
  =
  \left(
    \Delta t,\;N_{\mathrm{sub}}=1,\;W_{\max},\;\dot W_{\max},\;G_{W,\max},\;
    f_{\max},\;\mathcal B_{\mathrm{area/affine/cov}}
  \right)
  \label{eq:minimal-operating-envelope}
\end{equation}
로 package/run manifest에 저장한다. Core wind는
\(\|W\|\leq W_{\max}\), \(\|\partial_tW\|\leq\dot W_{\max}\),
\(\|\nabla_xW\|\leq G_{W,\max}\) 안에서만 정량 주장하며,
force와 area/affine/covariance clamp bound 및 clamp-rate cap도 함께 동결한다.
\(f_{\max}\)는 anchor별 \(\|f_{\mathrm{aero},i}\|_2\)의 hard bound다.
```

```latex
\begin{equation}
  f_{\mathrm{aero},i}^t
  =
  \frac12\rho_{\mathrm{air}}a_i^{t-1}
  \left[
    C_n|v_{n,i}^t|v_{n,i}^tn_i^{t-1}
    +C_\tau\|v_{\tau,i}^t\|v_{\tau,i}^t
  \right].
  \label{eq:minimal-aero}
\end{equation}

모든 anchor에서 계산하며 hyper-reduction하지 않는다.
Zero relative wind에서 force가 0이고, normal flip/two-sided convention과
finite-force clamp가 unit test를 통과해야 한다.
```

### 반례 또는 차원 분석

Raw force가

\[
 \widetilde f=(f_{\max},f_{\max},0)^\top
\]

이라고 하자. Componentwise clip을 사용하면 각 성분은 bound 안이지만

\[
 \|\widetilde f\|_2=\sqrt2 f_{\max}>f_{\max}
\]

이므로 source가 선언한 anchor별 Euclidean norm hard bound를 위반한다. 반면 norm-preserving clip은

\[
 f=f_{\max}\frac{\widetilde f}{\|\widetilde f\|_2}
\]

로 방향을 보존하면서 정확히 \(\|f\|_2=f_{\max}\)를 만족한다.

Aero raw force의 차원은

\[
 [\rho_{\mathrm{air}}a v^2]
 =\frac{M}{L^3}L^2\frac{L^2}{T^2}
 =\frac{ML}{T^2}
\]

이므로 \(f_{\max}/\|\widetilde f\|_2\)는 무차원이다.

또한 \(f_{\max}\)는 dynamics 결과에 적용되는 runtime bound이므로 wind input처럼 사전 input pre-check로 판정할 수 없다. Raw-force clip rate를 결과를 본 뒤 OOD로 소급 분류해서도 안 된다.

### 판정

**수용.** Hard norm clamp와 clamp-rate policy를 명시해야 한다. Smooth saturation이나 same-frame corrector는 visible jerk가 실제 관찰되기 전에는 추가하지 않는다.

### 허용되는 최소 replacement LaTeX

`eq:minimal-aero` 전체를 동일 label로 다음과 같이 교체한다.

```latex
\begin{equation}
\begin{aligned}
  \widetilde f_{\mathrm{aero},i}^t
  &=
  \frac12\rho_{\mathrm{air}}a_i^{t-1}
  \left[
    C_n|v_{n,i}^t|v_{n,i}^tn_i^{t-1}
    +C_\tau\|v_{\tau,i}^t\|_2v_{\tau,i}^t
  \right],\\
  f_{\mathrm{aero},i}^t
  &=
  \begin{cases}
    \widetilde f_{\mathrm{aero},i}^t,
    &\|\widetilde f_{\mathrm{aero},i}^t\|_2\leq f_{\max},\\[0.35em]
    f_{\max}
    \dfrac{\widetilde f_{\mathrm{aero},i}^t}
          {\|\widetilde f_{\mathrm{aero},i}^t\|_2},
    &\|\widetilde f_{\mathrm{aero},i}^t\|_2>f_{\max}.
  \end{cases}
\end{aligned}
  \label{eq:minimal-aero}
\end{equation}
```

그 뒤 다음 trace와 failure policy를 추가한다.

```latex
\[
  c_{f,i}^t
  =\mathbb I\!\left[
    \|\widetilde f_{\mathrm{aero},i}^t\|_2>f_{\max}
  \right],\qquad
  r_{f,\mathrm{clip}}
  =\frac{\sum_{i,t}c_{f,i}^t}
         {N_{\mathrm{anchor\text{-}frame}}^{\mathrm{required}}}.
\]
```

```latex
\(f_{\max}>0\)는 input-OOD pre-check가 아니라 finite raw force에 적용하는
runtime Euclidean-norm bound다. Non-finite raw force는 clamp하지 않고 즉시
in-envelope frame failure로 처리한다. Development에서 동결한
\(\tau_{f,\mathrm{clip}}\)에 대해
\(r_{f,\mathrm{clip}}>\tau_{f,\mathrm{clip}}\)이면 해당 canonical sequence는
in-envelope method failure다.
```

Operating-envelope 설명에는 다음 경계를 추가한다.

```latex
Input pre-check는 \(\Delta t,N_{\mathrm{sub}},W,\partial_tW,\nabla_xW\)와
run 전에 알려진 typed preset만 검사한다. Runtime force/clamp-rate 위반을
관측 뒤 declared OOD로 재분류하지 않는다.
```

### Codex action

- Raw force는 `f_aero_raw`, bounded force는 `f_aero`로 분리한다.
- Clamp는 최종 3-vector의 Euclidean norm에 한 번만 적용한다.
- Componentwise clamp, axis별 clamp와 normal/tangent channel별 독립 clamp를 만들지 않는다.
- Non-finite raw force에 0 또는 `f_max`를 대입하지 않는다.
- Existing selector와 coupled solve는 bounded `f_aero`만 소비한다.

### Acceptance test

- [ ] Zero relative wind에서 raw/bounded force가 정확히 0이다.
- [ ] `||raw|| <= f_max`이면 bounded force가 raw force와 bitwise 또는 tolerance 내 동일하다.
- [ ] `||raw|| > f_max`이면 `||bounded|| = f_max`이며 방향 cosine이 1이다.
- [ ] `n -> -n`에서 raw와 bounded force가 모두 불변이다.
- [ ] Non-finite raw force가 fail-fast되고 clip counter에 들어가지 않는다.
- [ ] SI/nd 변환 뒤 force와 clip decision이 parity tolerance 내 일치한다.
- [ ] `r_f_clip`과 first-failure reason이 trace에 저장된다.

### Minimal V0 scope 영향

- 기존 aero law 뒤 vector norm clamp 한 건
- existing trace에 counter/rate 한 건
- 새 force model, solver, corrector, network 또는 substep 없음

---

## FB-03 - V0-R1 pre-check의 `nu_decode` 누락

- **PRIORITY:** `PATCH BEFORE V0-R1`
- **판정:** **수용**
- **Equation label:** `eq:minimal-r1-version-contract`
- **Source line:** `1411--1418; 1537--1540`

### 원문의 LaTeX 전사

```latex
Topology distillation은 runtime network가 아니라 setup용 V0-R1 module 하나다. 그 실행 계약을
\begin{equation}
  \mathcal V_{\mathrm{R1}}
  =(\nu_{\mathrm{anchor}},\nu_{\mathrm{cand}},\nu_{\mathrm{feat}},
    \nu_{\mathrm{arch}},\nu_{\mathrm{label}},\nu_{\mathrm{loss}},
    \nu_{\mathrm{decode}},\nu_{\mathrm{split}})
  \label{eq:minimal-r1-version-contract}
\end{equation}
```

Training-entry pre-check 원문은 다음과 같다.

```latex
  \item 각 training entry 전에
  \(\nu_{\mathrm{anchor}},\nu_{\mathrm{cand}},\nu_{\mathrm{feat}},\nu_{\mathrm{arch}},
  \nu_{\mathrm{label}},\nu_{\mathrm{loss}},\nu_{\mathrm{split}}\), decoder schema, seed와 training data/label
  hash가 non-placeholder인 training-static config인지 검사하고 하나라도 비었으면 fail-fast한다.
```

### 반례

`nu_decode`가 placeholder인 상태에서 `decoder schema` 문자열만 존재하면 training entry가 통과할 수 있다. 그러나 같은 model output에 대해

\[
 \tau_{\mathrm{rel}}=0.4
 \qquad\text{와}\qquad
 \tau_{\mathrm{rel}}=0.8
\]

은 accepted graph, retained degree, package rejection rate와 Gate A response를 바꾼다. `nu_decode`가 비어 있으면 model/config hash만으로 predicted scaffold 결과를 재현할 수 없다.

### 판정

**수용.** Manifest validation field 한 개의 누락이다.

### 허용되는 최소 replacement LaTeX

Lifecycle step 2의 field list만 다음과 같이 바꾼다.

```latex
\item 각 training entry 전에
\(\nu_{\mathrm{anchor}},\nu_{\mathrm{cand}},\nu_{\mathrm{feat}},\nu_{\mathrm{arch}},
\nu_{\mathrm{label}},\nu_{\mathrm{loss}},\nu_{\mathrm{decode}},
\nu_{\mathrm{split}}\), decoder schema, seed와 training data/label
hash가 non-placeholder인 training-static config인지 검사하고
하나라도 비었으면 fail-fast한다.
```

`decoder schema`가 `nu_decode` 내부 field라면 중복 표기를 제거할 수 있지만, 그 정리는 필수가 아니다.

### Acceptance test

- [ ] `nu_decode=PLACEHOLDER` 또는 누락이면 training entry가 fail-fast된다.
- [ ] `tau_rel`, `tau_layer`, degree/radius cap 또는 tie-break가 바뀌면 `nu_decode` hash가 바뀐다.
- [ ] 같은 model weight라도 decoder hash가 다르면 동일 result의 보정으로 취급하지 않는다.
- [ ] Test payload open 이전에 모든 `V_R1` field가 non-placeholder다.

### Minimal V0 scope 영향

- manifest validation field 한 개
- network, loss, decoder algorithm, threshold sweep와 runtime 변화 없음

---

## FB-04 - Privileged teacher counterfactual base state/rollout 정의 누락

- **PRIORITY:** `PATCH BEFORE V0-03 / GATE B`
- **판정:** **후속 보류**
- **Equation label:** 없음; Gate B source paragraph
- **Source line:** `1587--1606`

### 원문의 LaTeX 전사

```latex
\[
\begin{aligned}
  &\text{strong Global rank sweep}\qquad\text{vs.}\\
  &\text{Global + privileged-teacher-ranked}\\
  &\text{complementary Local}
\end{aligned}
\]

을 비교한다. Privileged-teacher-ranked Local이 tip flutter, traveling gust, velocity trajectory 또는
target-band PSD에서 식~\eqref{eq:minimal-pareto-effects}의 clustered effect-margin improvement를
반복적으로 만들지 못하면
MC2와 selector 개발을 중단하고 Global-only 방향으로 축소한다.

Privileged-teacher-ranked selector는 setup에서 동결한 동일 atomic units와 \(K_A,K_D\),
동일 coupled solver를 사용하되,
privileged teacher의 다음 \(H\) frame velocity/target-band error 감소량으로 unit을 rank한다.
Future label search 비용은 runtime latency에서 제외하지만 선택 뒤 실제 execution 비용은 포함한다.
\(H\), error metric과 tie-break를 development split에서 한 번 동결한다.
이는 per-unit ranking으로 Local 필요성을 빠르게 판정하는 scope-control diagnostic이며,
cross-unit coupling과 redundancy를 탐색하는 exact best-subset oracle이나 formal upper bound가 아니다.
```

### 반례

현재 Decay unit \(D\)가 있는 상태에서 candidate \(A,B\)를 평가한다고 하자. Global-only를 base로 쓰면

\[
 \Delta_A^{(G)}=5,\qquad \Delta_B^{(G)}=4
\]

여서 \(A\)가 먼저지만, 동일 persistent state와 Decay \(D\)를 포함한 base에서는 cross coupling으로

\[
 \Delta_A^{(G+D)}=-1,\qquad \Delta_B^{(G+D)}=3
\]

이 될 수 있다. Candidate slot을 zero-reset하는 경우와 기존 Decay state를 유지하는 경우도 ranking이 달라진다. 두 구현 모두 현재의 “다음 \(H\) frame error 감소량” 문장을 만족할 수 있으므로 teacher result가 유일하지 않다.

### 판정

**후속 보류.** V0-00/V0-01을 막지는 않지만 V0-03 teacher-ranked Local과 Gate B 실행 전에 common state, base control과 rollout rule을 동결해야 한다.

### 허용되는 최소 추가 LaTeX

Gate B의 teacher 설명 뒤에 다음 계약을 추가한다.

```latex
Privileged-teacher counterfactual version은
\(\texttt{teacher\_counterfactual\_v1}\)로 고정한다. 모든 candidate rollout은
동일한 common persistent state
\[
  s_{\mathrm{cf}}^{t-1}
  =\left(
    q^{t-1},\dot q^{t-1},
    z_{\mathrm{sup}}^{t-1},\dot z_{\mathrm{sup}}^{t-1},
    \mathcal A_{t-1},\mathcal D_{t-1}
  \right)
\]
와 동일한 wind/reference horizon, solver, precision 및 seed에서 시작한다.
```

```latex
\(\mathcal A_t^{\mathrm{mand}}\)를 minimum dwell 때문에 유지해야 하는 Active set이라 한다.
\(c_t^{\mathrm{base}}\)는 desired set을 \(\mathcal A_t^{\mathrm{mand}}\)로 둔 기존
frozen admission policy의 결과이고, \(c_t^{(r)}\)는 candidate \(r\)을
highest-priority optional unit으로 한 번 삽입한 같은 policy의 결과다.
Candidate는 stored slot state를 reset하지 않는다.
```

```latex
\begin{equation}
  \Delta_r^t
  =
  \mathcal E_H\!\left(c_t^{\mathrm{base}};s_{\mathrm{cf}}^{t-1}\right)
  -
  \mathcal E_H\!\left(c_t^{(r)};s_{\mathrm{cf}}^{t-1}\right),
  \label{eq:minimal-teacher-unit-benefit}
\end{equation}
```

```latex
여기서 \(\mathcal E_H\)는 development에서 동결한 velocity/target-band teacher metric이다.
각 counterfactual에서는 frame \(t\)의 admission 결과만 다르게 하고,
그 결과의 Active desired set을 \(H\) frame 동안 hold한다.
Decay/release는 기존 frozen rule을 따르며 horizon 안에서 teacher ranking을 재귀 호출하지 않는다.
Teacher desired set은 \(\Delta_r^t\)의 고정 tie-break TopK로 만들고,
첫 unit 선택 뒤 benefit을 재계산하지 않는다. 따라서 exact subset oracle이 아니다.
```

선택 set을 명시하려면 다음을 사용한다.

```latex
\[
  \mathcal A_t^{\mathrm{teacher,des}}
  =\mathcal A_t^{\mathrm{mand}}
  \cup
  \operatorname{TopK}_{K_A-|\mathcal A_t^{\mathrm{mand}}|}
  \left(\{\Delta_r^t\}_{r\in\mathcal C_t}\right).
\]
```

### Codex action

- `teacher_counterfactual_v1`은 offline diagnostic code에만 추가한다.
- Candidate rollout 시작 시 full persistent state를 deep-copy한다.
- Decay state를 zero-reset하지 않는다.
- Candidate가 admission capacity 때문에 실제로 들어가지 못하면 base와 동일한 rollout이므로 score 0이 되어야 한다.
- Subset enumeration, greedy re-score, learned teacher와 runtime network를 추가하지 않는다.

### Acceptance test

- [ ] 모든 candidate rollout의 initial state hash, wind hash, reference hash와 solver config가 동일하다.
- [ ] 동일 input/seed에서 `Delta_r`와 ranking이 재현된다.
- [ ] Candidate의 stored Active/Decay state가 reset되지 않는다.
- [ ] Admission되지 않은 candidate의 rollout이 base와 동일하고 `Delta_r=0`이다.
- [ ] Candidate order tie는 stable ID rule로 결정된다.
- [ ] `K_A`, `K_D`, coupled solver와 actual execution latency accounting이 기존 Gate B와 동일하다.
- [ ] Exact subset enumeration 또는 horizon 내 recursive teacher ranking이 없다.

### Minimal V0 scope 영향

- existing offline teacher의 comparison contract만 동결
- runtime selector, solver, state machine와 latency path 변화 없음
- combinatorial oracle, learned gate 또는 새 module 없음

---

## FB-05 - 이전 반영 사항 및 회귀 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용 / 감수 유지**

| 항목 | 위치 | 판정 |
|---|---|---|
| Privileged-teacher-ranked 명칭과 non-oracle boundary | source `1587--1606` | 수용 |
| Area head `zeta_i^a=g_{theta,a}(h_i)` | `eq:minimal-r1-area-decode`, `1461--1476` | 수용 |
| Dimensionless PSD와 `f_PSD` 표기 | `eq:minimal-log-psd-errors`, `1700--1726` | 수용 |
| Corotated tangent-plane bending | `eq:minimal-bending-corotated-frame/projector` | 수용 |
| Attachment line normalization | `eq:minimal-attachment-line-gauge` | 수용 |
| Fixed-Hessian package mode claim boundary | `eq:minimal-global-basis` 주변 | 감수 |
| Numerical-rank semantics | `eq:minimal-tangent-numerical-rank` | 수용 |
| Affine-safe SVD/reflection/scale rule | `eq:minimal-affine-safe-map` | 수용 |

### 원문 확인 예시

```latex
Area head는 node embedding \(h_i\)만 소비하는 scalar head이며 global pooling이나 추가
message-passing stage를 사용하지 않는다. Dimensionless floor
\(\varepsilon_{a,\mathrm{raw}}>0\)에 대해
\begin{equation}
\begin{aligned}
  \zeta_i^a
  &=g_{\theta,a}(h_i),\\
  \widetilde a_i
  &=\operatorname{softplus}(\zeta_i^a)+\varepsilon_{a,\mathrm{raw}},\\
  a_i^0
  &=A_{\mathrm{ref}}\frac{\widetilde a_i}{\sum_{k\in V}\widetilde a_k}.
\end{aligned}
  \label{eq:minimal-r1-area-decode}
\end{equation}
로 \(\lvert V\rvert\times1\) scalar output과 positive total ownership을 construction으로 만족한다.
Area-head width/depth/activation과 output field ID는 \(\nu_{\mathrm{arch}}\)가 소유한다.
```

```latex
를 계산한다. 이 절의 \(f_{\mathrm{PSD}}\)는 temporal frequency다.
PSD는 per-bin power가 아니라 one-sided velocity-density PSD convention으로 고정하며
\([P]=L^2/T\)다. Sampling rate, window normalization, DC/Nyquist 처리와 development에서 동결한
frequency bin \(B\)를 evaluation hash에 저장한다. PSD와 같은 unit을 가진
\(\varepsilon_P>0\)를 사용하고, unit-parity를 나타내는 기존 typed conversion scale
\((L_0,T_0)\)에 대해
\begin{equation}
\begin{aligned}
  \widehat f_{\mathrm{PSD}}&=f_{\mathrm{PSD}}T_0,&
  \widehat\varepsilon_P&=\varepsilon_P\frac{T_0}{L_0^2},\\
  \widehat P_s(\widehat f_{\mathrm{PSD}})
  &=P_s(f_{\mathrm{PSD}})\frac{T_0}{L_0^2},\\
  \widehat P_{\mathrm{ref},s}(\widehat f_{\mathrm{PSD}})
  &=P_{\mathrm{ref},s}(f_{\mathrm{PSD}})\frac{T_0}{L_0^2},\\
  \delta_s(\widehat f_{\mathrm{PSD}})
  &=\log\!\left(\widehat P_s(\widehat f_{\mathrm{PSD}})+\widehat\varepsilon_P\right)
    -\log\!\left(\widehat P_{\mathrm{ref},s}(\widehat f_{\mathrm{PSD}})+\widehat\varepsilon_P\right),\\
  e_{\mathrm{PSD,RMS},s}
  &=\left(\frac{1}{|\widehat B|}
    \sum_{\widehat f_{\mathrm{PSD}}\in\widehat B}
    \delta_s(\widehat f_{\mathrm{PSD}})^2\right)^{1/2},\\
  e_{\mathrm{PSD,L1},s}
  &=\frac{1}{|\widehat B|}
    \sum_{\widehat f_{\mathrm{PSD}}\in\widehat B}
    |\delta_s(\widehat f_{\mathrm{PSD}})|.
\end{aligned}
  \label{eq:minimal-log-psd-errors}
```

### 판정 및 Minimal V0 scope 영향

위 항목은 변경하지 않는다. 회귀 test만 수행한다. 특히 fixed-Hessian basis를 exact physical tangent mode로 다시 바꾸거나, bending/transport backend를 재설계하지 않는다.

# 4. Patch 적용 후 acceptance checklist

## 4.1 Source/LaTeX

- [ ] Source SHA mismatch 시 자동 patch가 중단된다.
- [ ] 기존 47개 label 중 삭제/중복이 없다.
- [ ] 새 `eq:minimal-teacher-unit-benefit`을 추가한 경우 모든 reference가 유효하다.
- [ ] XeLaTeX가 2-pass fatal error 없이 완료된다.
- [ ] `.bib` 미존재에 따른 citation warning 이외의 undefined equation/reference가 없다.

## 4.2 Milestone gate

- [ ] V0-01 전에 `mass_matrix_type=lumped_anchor_v1`이 package에 존재한다.
- [ ] V0-02 전에 raw/bounded aero force와 force-clamp trace가 존재한다.
- [ ] V0-R1 training 전에 `nu_decode` non-placeholder validation이 존재한다.
- [ ] V0-03/Gate B 전에 `teacher_counterfactual_v1` common-state replay가 존재한다.

## 4.3 금지된 확장 부재

- [ ] 새 shell/hinge backend 없음
- [ ] KKT/Schur 없음
- [ ] same-frame aero corrector 없음
- [ ] learned selector/gate 없음
- [ ] force hyper-reduction 없음
- [ ] additional quantitative asset class 없음
- [ ] exact subset oracle 없음

# 5. CG 논문 관점의 영향

네 수정은 novelty를 높이기 위한 새 기능이 아니다. 다음 claim을 구현과 evidence가 실제로 지지하도록 만드는 재현성 계약이다.

- `FB-01`: Global/Local basis, selector와 mass-norm metric이 구현체마다 달라지는 것을 방지한다.
- `FB-02`: visible explosion 방지용 clamp가 논문에서 선언한 Euclidean hard bound와 일치하도록 한다.
- `FB-03`: MC1 predicted scaffold의 decoder leakage와 test-aware retuning 가능성을 차단한다.
- `FB-04`: Gate B의 privileged-teacher gap을 재현 가능한 diagnostic으로 만든다.

따라서 이 패치 후에도 Accept를 결정하는 핵심은 그대로다.

\[
 \text{Predicted scaffold}\approx\text{Oracle scaffold}
 >\text{oriented-kNN},
\]

\[
 \text{Global + privileged-teacher-ranked Local}
 \text{가 strong Global-rank Pareto envelope를 확장하는가},
\]

\[
 \text{deterministic selected Local이 그 이득을 실제 p95에서 유지하는가}.
\]

# 6. 최종 결론

현재 Minimal V0의 구현 스코프와 method architecture는 유지한다. 다음 네 계약만 milestone 전에 닫는다.

1. `M = blkdiag(m_i I_3)`
2. raw aero force에 대한 Euclidean norm clamp와 rate policy
3. V0-R1 pre-check의 `nu_decode`
4. privileged teacher의 common counterfactual state와 per-unit score

그 외 수식과 backend는 변경하지 않는다. 네 항목이 반영되면 canonical source는 V0-01--V0-03 및 V0-R1 구현을 위한 재현 가능한 contract로 동결 가능하다.

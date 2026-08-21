---
title: "Minimal V0 반영본 최종 확인 및 Codex 잔여 패치 명세"
document_type: "codex_final_verification_patch_spec"
date: "2026-08-20"
source_file: "붙여넣은 텍스트 (1).txt"
source_sha256: "a0c8ae37ea233456fc8dee2bf20e9e2885a77ebcf20575c5064a6f222067aa29"
scope_policy: "No new solver/module unless Gate failure or observed visual artifact requires it"
---

# Minimal V0 반영본 최종 확인 및 Codex 잔여 패치 명세

## 0. 문서 역할

이 파일은 현재 canonical source의 반영 상태를 검증한 보고서이자 VS Code Codex extension에 직접 입력할 수 있는 최소 패치 명세다. Equation label을 1차 기준으로 사용하고 source line은 위 SHA-256에만 유효한 보조 기준으로 사용한다.

### 절대 스코프 규칙

- 새 solver, shell backend, runtime network, corrector, KKT/Schur, hyper-reduction, 추가 asset class 또는 병렬 mainline을 만들지 않는다.
- `KEEP / NO PATCH` 및 `감수` 항목은 코드나 수식을 변경하지 않는다.
- `Oracle scaffold`와 `Oracle flat-rest scaffold`는 selector 명칭이 아니므로 유지한다.
- 관련 없는 formatting, 변수명, 파일 구조, threshold 또는 config를 정리하지 않는다.
- 기존 equation label을 유지한다.
- patch 뒤 XeLaTeX compile, label/reference check와 각 acceptance test를 실행한다.

## 1. 종합 판정

현재 source에서 새 solver/module을 요구하는 치명적 수식 오류는 발견되지 않았다. 이전 P0 및 contract 보완은 정상 반영되었으며 Minimal V0 구현 동결은 **GO**다. 남은 실제 패치는 다음 세 개뿐이다.

1. `FV-11`: selector 명칭을 `Oracle-selected`에서 `privileged-teacher-ranked`로 축소한다.
2. `FV-12`: area raw logit을 `zeta_i^a = g_{theta,a}(h_i)`로 명시한다.
3. `FV-13`: PSD 절의 temporal frequency를 `f`에서 `nu`로 변경한다.

`FV-10`의 실제 V0-R1 config 값은 training entry 전에 동결해야 하지만, Oracle scaffold 기반 V0-01--04를 막지 않는다.

| ID | 항목 | Priority | 판정 | 적용 시점 |
|---|---|---|---|---|
| FV-01 | Corotated tangent-plane bending projector 반영 확인 | KEEP / NO PATCH | 수용 | V0-01 유지 |
| FV-02 | Attachment-line eigenvector unit normalization 반영 확인 | KEEP / NO PATCH | 수용 | V0-01 유지 |
| FV-03 | SI--nondimensional map과 release proxy 차원 반영 확인 | KEEP / NO PATCH | 수용 | V0-00 및 V0-03 유지 |
| FV-04 | Fixed-Hessian package mode의 claim boundary 반영 확인 | KEEP / NO PATCH | 감수 | V0-01 유지 |
| FV-05 | Area total normalization convention 반영 확인 | KEEP / NO PATCH | 수용 | V0-01 및 V0-R1 유지 |
| FV-06 | Shared numerical-rank semantics 반영 확인 | KEEP / NO PATCH | 수용 | V0-01 및 V0-04 유지 |
| FV-07 | Layer-label positive-class polarity 반영 확인 | KEEP / NO PATCH | 수용 | V0-R1 유지 |
| FV-08 | Affine-safe SVD, reflection 및 scale clamp 반영 확인 | KEEP / NO PATCH | 수용 | V0-04 유지 |
| FV-09 | Dimensionless log-PSD metric 반영 확인 | KEEP FORMULA / PATCH NOTATION IN FV-13 | 수용 | 평가 스크립트 유지 |
| FV-10 | V0-R1 canonical config freeze lifecycle 확인 | FOLLOW-UP HOLD / BLOCK TRAINING UNTIL SET | 후속 보류 | V0-R1 training entry 직전 |
| FV-11 | Oracle-selected Local 명칭의 과도한 upper-bound claim | PATCH WORDING NOW | 반박 (명칭만) | V0-03/Gate B 문서 동결 전 |
| FV-12 | Area raw logit의 생성식 누락 | PATCH BEFORE V0-R1 | 수용 | V0-R1 architecture freeze 전 |
| FV-13 | Force와 temporal frequency의 f 표기 충돌 | PATCH NOTATION BEFORE EVALUATION FREEZE | 수용 | Gate A/B evaluation code 동결 전 |

## 2. Codex 실행 순서

1. `FV-11`의 selector-context 명칭만 교체한다. `Oracle scaffold` 관련 문자열은 건드리지 않는다.
2. `FV-12`의 `eq:minimal-r1-area-decode`를 동일 label로 교체한다.
3. `FV-13`의 PSD frequency symbol만 `nu`로 교체한다. Force mapping은 변경하지 않는다.
4. `FV-01`--`FV-09`는 regression test만 수행하고 수식/코드를 변경하지 않는다.
5. `FV-10`은 training entry validation만 유지하고 actual config 선택 전까지 predicted-scaffold training을 시작하지 않는다.

## 3. 상세 리뷰

## FV-01 - Corotated tangent-plane bending projector 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-01 유지
- **Equation label:** `eq:minimal-bending-corotated-frame; eq:minimal-bending-corotated-projector`
- **Source line:** `482--526`

### 원문 LaTeX 전사

```latex
\begin{equation}
\begin{aligned}
  \mathcal R_i(J)
  &=\left[
    \frac{J(:,1)}{\|J(:,1)\|_2},\;
    n_i(J)\times\frac{J(:,1)}{\|J(:,1)\|_2},\;
    n_i(J)
  \right],\\
  n_i(J)
  &=s_i\frac{J(:,1)\times J(:,2)}{\|J(:,1)\times J(:,2)\|_2},\\
  R_i^0&=\mathcal R_i(J_i^0),\qquad
  R_i^{t,\mathrm{pred}}=\mathcal R_i(J_i^{t,\mathrm{pred}}),\qquad
  Q_i^t=R_i^{t,\mathrm{pred}}(R_i^0)^\top .
\end{aligned}
  \label{eq:minimal-bending-corotated-frame}
\end{equation}
Stored \((t_{i1}^0,t_{i2}^0,n_i^0)\)와 fitted rest frame이 정확히 같다고 가정하지 않는다.
Rest와 predictor에 같은 construction을 사용하므로 rest에서 \(Q_i^0=I\)이고 proper rigid transform에서
\(R_i^{t,\mathrm{pred}}=QR_i^0\)다. 기존 tangent-fit rank/condition/normal floor를 그대로 적용하며
실패를 임의 frame fallback으로 우회하지 않는다.

Fitted normal을
\(n_{i,\mathrm{fit}}^0=R_i^0(:,3)\),
\(n_i^{t,\mathrm{pred}}=R_i^{t,\mathrm{pred}}(:,3)\)라 하고
\begin{equation}
\begin{aligned}
  b_{ia}^0
  &=\bigl((n_{i,\mathrm{fit}}^0)^\top d_{ia}^0\bigr)n_{i,\mathrm{fit}}^0,
  &b_{ia}^t&=Q_i^t b_{ia}^0,\\
  \mathcal C_{ia}^{\mathrm{bend},t}
  &=\{b_{ia}^t+v:(n_i^{t,\mathrm{pred}})^\top v=0\},
  &\Pi_{ia}^{\mathrm{bend},t}(d)
  &=d-n_i^{t,\mathrm{pred}}(n_i^{t,\mathrm{pred}})^\top(d-b_{ia}^t).
\end{aligned}
  \label{eq:minimal-bending-corotated-projector}
\end{equation}
```

### 반례 또는 차원 분석

Frozen rest frame에서 \(n=n_{i,\mathrm{fit}}^0\), \(b=(n^\top d_{ia}^0)n\)로 두고 normal perturbation을 \(d(\epsilon)=d_{ia}^0+\epsilon n\)으로 잡는다. 그러면 \(n^\top(d_{ia}^0-b)=0\)이므로

\[
\Pi(d(\epsilon))=d_{ia}^0,\qquad
 d(\epsilon)-\Pi(d(\epsilon))=\epsilon n,
\]

이고 minimized quadratic energy는

\[
E(\epsilon)=\frac12 w_{ia}^{\mathrm{bend}}\epsilon^2,
\qquad
\left.\frac{d^2E}{d\epsilon^2}\right|_{\epsilon=0}
=w_{ia}^{\mathrm{bend}}>0.
\]

따라서 이전 nonzero-\(d_{ia}^0\) sphere orbit의 \(O(\epsilon^4)\) normal-stiffness 퇴화가 제거되었다. Proper rigid transform \(Q\in SO(3)\)에서도 \(d'=Qd_{ia}^0\), \(n'=Qn\), \(b'=Qb\)이므로 \(\Pi'(Qd_{ia}^0)=Qd_{ia}^0\)가 성립한다.

### 판정 및 Codex action

현재 수식과 projector를 변경하지 않는다. Unit fixture와 integrated fixture만 유지한다.

### Acceptance test

- [ ] Rest에서 Q_i^0 = I이고 projector residual이 0인지 확인한다.
- [ ] Frozen-frame central finite difference normal stiffness가 w_bend와 허용오차 내 일치하는지 확인한다.
- [ ] Random proper rigid transform reproduction을 확인한다.
- [ ] Integrated normal perturbation에서 positive non-collapsed response를 확인한다.

### Minimal V0 scope 영향

새 constraint, solver, local/global iteration 또는 runtime stage가 없다.

## FV-02 - Attachment-line eigenvector unit normalization 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-01 유지
- **Equation label:** `eq:minimal-attachment-line-gauge; eq:minimal-local-line-gauge`
- **Source line:** `384--415`

### 원문 LaTeX 전사

```latex
\begin{equation}
  C_{\mathrm{att}}g_{\mathrm{att}}=\lambda_1g_{\mathrm{att}},\qquad
  \|g_{\mathrm{att}}\|_2=1,\qquad
  L_{\mathrm{att}}=g_{\mathrm{att}}g_{\mathrm{att}}^\top,\qquad
  \frac{\lambda_1-\lambda_2}{\lambda_1+\varepsilon_{\mathrm{att}}}
  \geq\tau_{\mathrm{gap}}>0
  \label{eq:minimal-attachment-line-gauge}
\end{equation}
를 요구한다. 물리적 gauge는 vector \(g_{\mathrm{att}}\)가 아니라 sign-invariant unoriented line
\(L_{\mathrm{att}}\)다. Eigensolver 출력은 \(q_i\)와 \(L_{\mathrm{att}}\)를 계산하기 전에
unit-normalize한다. 따라서 \(L_{\mathrm{att}}\)는
\(L_{\mathrm{att}}^2=L_{\mathrm{att}}\), \(\operatorname{tr}(L_{\mathrm{att}})=1\)인
rank-one orthogonal projector이고, eigenvector의 임의 scale과 sign에 무관하다.
\(\lambda_1\) floor나 eigengap을 통과하지 못하면 world-axis에 맞춘 임의 sign 또는
secondary direction으로 우회하지 않고 package를 reject한다.

Anchor \(i\)에서는 \(P_i^0=I-n_i^0(n_i^0)^\top\),
\(q_i=P_i^0g_{\mathrm{att}}\)에 대해 \(\|q_i\|\geq\tau_{\mathrm{gauge}}\)를 요구하고
\begin{equation}
  L_i^0=\frac{q_iq_i^\top}{q_i^\top q_i},\qquad
  t_{i1}^0\in\operatorname{range}(L_i^0),\quad \|t_{i1}^0\|_2=1,\qquad
  t_{i2}^0=n_i^0\times t_{i1}^0
  \label{eq:minimal-local-line-gauge}
\end{equation}
로 둔다. \(t_{i1}^0\mapsto-t_{i1}^0\)와 \(t_{i2}^0\mapsto-t_{i2}^0\)는 같은 gauge다.
Opposite pair, feature, label, loss와 decoder는 이 동시 sign flip에 불변이어야 한다.
Byte-level serialization에 representative가 필요하면 immutable attachment-ID 순서에서 처음 만나는
nondegenerate attachment pair에만 sign을 맞추며, world-axis component sign은 사용하지 않는다.
Minimal V0의 learned in-plane correction은 \(\Delta\theta_i=0\)으로 고정하고 topology model 출력에
추가하지 않는다. Gauge algorithm, attachment IDs/weights, eigengap/floor, representative rule을
normalization/projector tolerance와 함께
\(\texttt{tangent\_gauge\_version=attachment\_line\_v1}\) package hash에 저장한다.
```

### 반례 또는 차원 분석

Eigenvector는 scale에 대해 유일하지 않으므로 unit normalization이 없다면 \(g'_{\mathrm{att}}=c g_{\mathrm{att}}\)에 대해

\[
L'_{\mathrm{att}}=c^2L_{\mathrm{att}},\qquad
\|P_i^0g'_{\mathrm{att}}\|_2=|c|\,\|P_i^0g_{\mathrm{att}}\|_2
\]

가 되어 동일 geometry의 package acceptance가 eigensolver 출력 scale에 의존한다. 현재 source는 \(\|g_{\mathrm{att}}\|_2=1\)을 equation에 포함하고, \(q_i\)와 \(L_{\mathrm{att}}\) 계산 전에 normalize하도록 명시한다. 따라서

\[
L_{\mathrm{att}}^2=L_{\mathrm{att}},\qquad
\operatorname{tr}(L_{\mathrm{att}})=1
\]

인 rank-one orthogonal projector가 되어 scale/sign ambiguity가 제거된다.

### 판정 및 Codex action

추가 패치 없이 유지한다.

### Acceptance test

- [ ] abs(||g_att||_2 - 1) <= tolerance.
- [ ] ||L_att^2 - L_att||_F 및 |tr(L_att)-1|가 tolerance 이하.
- [ ] raw eigenvector에 임의 scale/sign을 적용해도 normalize 후 package result가 동일.
- [ ] g_att -> -g_att에서 line feature와 decoder result가 동일.

### Minimal V0 scope 영향

정규화와 projector fixture는 기존 V0-01 내부 연산이며 runtime/module 수를 늘리지 않는다.

## FV-03 - SI--nondimensional map과 release proxy 차원 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-00 및 V0-03 유지
- **Equation label:** `eq:minimal-si-nd-typed-map; eq:minimal-rayleigh-unit-map; eq:minimal-decay-release`
- **Source line:** `195--214; 232--236; 1014--1050`

### 원문 LaTeX 전사

```latex
\begin{equation}
\begin{aligned}
  \widehat x&=x/L_0,&
  \widehat t&=t/T_0,&
  \widehat{\Delta t}&=\Delta t/T_0,\\
  \widehat m&=m/M_0,&
  \widehat a&=a/L_0^2,&
  \widehat\Sigma&=\Sigma/L_0^2,\\
  \widehat\rho_A&=\rho_A L_0^2/M_0,&
  \widehat\kappa_{\mathrm{str}}&=\kappa_{\mathrm{str}}T_0^2/M_0,&
  \widehat\kappa_{\mathrm{bend}}&=\kappa_{\mathrm{bend}}T_0^2/(M_0L_0^2),\\
  \widehat\rho_{\mathrm{air}}&=\rho_{\mathrm{air}}L_0^3/M_0,&
  \widehat W&=WT_0/L_0,&
  \widehat{\partial_tW}&=(\partial_tW)T_0^2/L_0,\\
  \widehat{\nabla_xW}&=T_0\nabla_xW,&
  \widehat f&=fT_0^2/(M_0L_0),&
  \widehat f_{\max}&=f_{\max}T_0^2/(M_0L_0).
\end{aligned}
  \label{eq:minimal-si-nd-typed-map}
\end{equation}

\begin{equation}
  \widehat\alpha=\alpha T_0,\qquad
  \widehat\beta=\frac{\beta}{T_0}
  \label{eq:minimal-rayleigh-unit-map}
\end{equation}

\begin{equation}
  \gamma_r^t
  =
  \max\!\left(0,1-\frac{t-t_{r,\mathrm{off}}}{N_{\mathrm{fade}}}\right),
  \qquad
  \widehat E_r^t
  =
  \frac{
    \tfrac12(\dot z_r^t)^\top M_{rr}^{\mathrm{loc}}\dot z_r^t
    +\tfrac12(z_r^t)^\top K_{rr}^{\mathrm{loc}}z_r^t
  }{E_{r,\mathrm{ref}}+\varepsilon_E},
  \quad
  \begin{aligned}
  M_{rr}^{\mathrm{loc}}&=\Psi_r^\top M\Psi_r,\\
  K_{rr}^{\mathrm{loc}}&=\Psi_r^\top K\Psi_r .
  \end{aligned}
  \label{eq:minimal-decay-release}
\end{equation}

여기서 \(\widehat E_r^t\)는 inter-patch/Global cross term을 소유하지 않는
scale-aware \emph{release proxy}이지 exact per-patch energy가 아니다.
Mass norm은 \(\|u\|_M:=\sqrt{u^\top Mu}\)로 정의한다. 따라서 displacement numerator의 단위는
\(\sqrt{M}L\), velocity numerator의 단위는 \(\sqrt{M}L/T\)이고, displacement/velocity proxy는
\[
  \widehat d_{x,r}^t
  =
  \frac{\|\Psi_rz_r^t\|_M}
       {\sqrt{M_{r,\mathrm{ref}}}L_{\mathrm{ref}}},
  \qquad
  \widehat d_{v,r}^t
  =
  \frac{\|\Psi_r\dot z_r^t\|_M}
       {\sqrt{M_{r,\mathrm{ref}}}V_{\mathrm{ref}}}
\]
로 고정한다.
분모에 \(M_{r,\mathrm{ref}}\) 자체를 곱하는 표현은 무차원이 아니고 asset mass scale에 의존하므로
Minimal V0 contract에 포함하지 않는다.
```

### 반례 또는 차원 분석

주요 항의 차원은 다음과 같이 소거된다.

\[
\left[\rho_A\frac{L_0^2}{M_0}\right]=1,\quad
\left[\kappa_{\mathrm{str}}\frac{T_0^2}{M_0}\right]=1,\quad
\left[\kappa_{\mathrm{bend}}\frac{T_0^2}{M_0L_0^2}\right]=1,
\]

\[
\left[f\frac{T_0^2}{M_0L_0}\right]=1,\qquad
[\alpha T_0]=1,\qquad
[\beta/T_0]=1.
\]

또한 mass norm \(\|u\|_M=\sqrt{u^\top Mu}\)는 displacement에 대해 \(\sqrt M L\), velocity에 대해 \(\sqrt M L/T\) 차원을 갖는다. 따라서 현재 분모

\[
\sqrt{M_{r,\mathrm{ref}}}L_{\mathrm{ref}},\qquad
\sqrt{M_{r,\mathrm{ref}}}V_{\mathrm{ref}}
\]

는 두 release proxy를 정확히 무차원화한다.

### 판정 및 Codex action

수식 변경 없이 typed conversion utility와 parity fixture를 유지한다.

### Acceptance test

- [ ] SI payload와 nondimensional payload의 trajectory/selector/release parity를 고정 tolerance로 검사한다.
- [ ] force, covariance, epsilon과 clamp가 unit map에 따라 함께 변환되는지 검사한다.
- [ ] release proxy가 asset mass 공통 scale 변화에 불변인지 검사한다.

### Minimal V0 scope 영향

추가 physics 또는 solver가 필요하지 않다.

## FV-04 - Fixed-Hessian package mode의 claim boundary 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **감수**
- **적용 시점:** V0-01 유지
- **Equation label:** `eq:minimal-structural-backend; eq:minimal-global-basis`
- **Source line:** `644--653; 682--700`

### 원문 LaTeX 전사

```latex
\begin{equation}
  E_{\mathrm{str}}^t(x)
  =
  \frac12\sum_{j\in\mathcal R}
  w_j\left\|A_jx-p_j^t\right\|_2^2,
  \qquad
  K=\sum_j w_jA_j^\top A_j,\quad
  h^t=\sum_j w_jA_j^\top p_j^t,\quad
  \bar h^t=h^t-Kx^0.
  \label{eq:minimal-structural-backend}

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
\end{equation}

여기서 \(K_f\)는 predictor-frozen one-local-step PD package가 저장한 fixed Hessian이다.
Minimized nonlinear constraint energy의 exact rest tangent라고 주장하지 않으며, 이 근사를 이유로
tangent-consistent basis, 새 shell backend 또는 추가 nonlinear iteration을 V0에 넣지 않는다.
```

### 반례 또는 차원 분석

Frozen plane normal \(n\)과 offset \(b\)에 대한 minimized constraint energy는

\[
E_{\min}(x)=\frac{w}{2}\bigl[n^\top(Ax-b)\bigr]^2,
\qquad
K_{\mathrm{tan}}=wA^\top nn^\top A.
\]

반면 one-local-step PD package가 저장하는 fixed Hessian은 \(K_{\mathrm{store}}=wA^\top A\)다. \(A\delta x=t\), \(n^\top t=0\)인 tangential perturbation에서는

\[
\delta^2E_{\min}=0,\qquad
\delta x^\top K_{\mathrm{store}}\delta x=w\|t\|_2^2>0.
\]

따라서 generalized eigenmodes는 exact physical rest-tangent modes가 아니다. 현재 source는 이미 ``fixed-Hessian package operator''라고 명명하고 exact rest tangent를 주장하지 않는다고 명시하므로 claim boundary가 올바르다.

### 판정 및 Codex action

Basis와 solver를 변경하지 않는다. 논문에서도 fixed-Hessian package modes라는 명칭을 유지한다.

### Acceptance test

- [ ] Global rank sweep이 동일 generalized_eigen_v1 prefix를 사용하는지 검사한다.
- [ ] Gate B에서 same-p95 quality를 strong Global rank sweep과 직접 비교한다.
- [ ] Visible over-stiffness가 실제 관찰되기 전 tangent-consistent basis를 추가하지 않는다.

### Minimal V0 scope 영향

새 shell backend, tangent-consistent eigenproblem 또는 nonlinear iteration을 추가하지 않는다.

## FV-05 - Area total normalization convention 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-01 및 V0-R1 유지
- **Equation label:** `eq:minimal-area-mass; eq:minimal-r1-area-decode`
- **Source line:** `585--607; 1461--1474`

### 원문 LaTeX 전사

```latex
\begin{equation}
  a_i^0>0,\qquad
  \sum_i a_i^0 \approx A_{\mathrm{asset}},\qquad
  m_i=\rho_A a_i^0,\qquad
  \sum_i m_i \approx M_{\mathrm{asset}}.
  \label{eq:minimal-area-mass}
\end{equation}

Canonical Minimal V0의 area total은 별도 calibrated surface-area estimator가 아니라
\[
  L_{\mathrm{ref}}=\operatorname{diameter}(x^0)>0,\qquad
  A_{\mathrm{ref}}=L_{\mathrm{ref}}^2,\qquad
  A_{\mathrm{asset}}:=A_{\mathrm{ref}},\qquad
  M_{\mathrm{asset}}:=\rho_AA_{\mathrm{ref}}
\]
인 deterministic package convention으로 고정한다. 이는 physical absolute surface-area 또는
engineering mass 측정값이라는 주장이 아니며, 목적은 Gaussian/anchor density가 달라도 relative
ownership과 motion scale이 크게 바뀌지 않게 하는 것이다. Object-normalized nondimensional geometry에서
\(L_{\mathrm{ref}}=1\)이면 \(A_{\mathrm{ref}}=1\)이다. 공통 area scale에 대한 trajectory invariance는
그 scale을 소유하는 \(M,C,K,\bar h,f_{\mathrm{aero}}\)가 함께 similarity-scale되고
관련 floor, epsilon과 clamp가 비활성이거나 같은 typed rule로 scale되는 조건에서만 요구한다.
Nondimensional branch의 \(a_i^0,\rho_A,m_i,A_{\mathrm{asset}},M_{\mathrm{asset}}\)는
\((L_0,M_0,T_0)\)로 정규화한 dimensionless package 값이다.

Area raw logit \(\zeta_i^a\)와 dimensionless floor
\(\varepsilon_{a,\mathrm{raw}}>0\)에 대해
\begin{equation}
  \widetilde a_i=\operatorname{softplus}(\zeta_i^a)+\varepsilon_{a,\mathrm{raw}},\qquad
  a_i^0=A_{\mathrm{ref}}\frac{\widetilde a_i}{\sum_{k\in V}\widetilde a_k}
  \label{eq:minimal-r1-area-decode}
\end{equation}
로 positive total ownership을 construction으로 만족한다.
여기서 unit-typed \(A_{\mathrm{ref}}>0\)는 물리적 절대 면적 추정값이 아니라
식~\eqref{eq:minimal-area-mass}의 deterministic 소유권 정규화 상수다.
Package build 전에 \(L_{\mathrm{ref}}=\operatorname{diameter}(x^0)>0\),
\(A_{\mathrm{ref}}=L_{\mathrm{ref}}^2\)로 고정하고, object-normalized nondimensional geometry에서
\(L_{\mathrm{ref}}=1\)이면 \(A_{\mathrm{ref}}=1\)이다. Training source mesh나 target static GS에
별도의 absolute-area calibration estimator를 만들지 않는다.
```

### 반례 또는 차원 분석

모든 area를 공통 상수 \(c>0\)로 scale하면 current V0 contract에서

\[
M'=cM,\quad C'=cC,\quad K'=cK,\quad
\bar h'=c\bar h,\quad f'_{\mathrm{aero}}=cf_{\mathrm{aero}}.
\]

따라서 equation 전체를 \(c\)로 나누면 trajectory는 이상적으로 동일하다. 차이가 생길 수 있는 지점은 absolute clamp/floor가 활성화되는 경우뿐이며 source가 typed scaling 조건을 명시한다. 현재 \(A_{\mathrm{ref}}=L_{\mathrm{ref}}^2\)는 physical surface-area estimator가 아니라 deterministic ownership normalization constant로 명시되어 있어 불필요한 calibration scope가 제거되었다.

### 판정 및 Codex action

현재 deterministic convention을 유지한다. Absolute area recovery claim을 추가하지 않는다.

### Acceptance test

- [ ] a_i^0 > 0 및 sum_i a_i^0 = A_ref를 tolerance 내 확인한다.
- [ ] 공통 area scale과 관련 typed clamp/floor를 함께 scale했을 때 trajectory parity를 검사한다.
- [ ] Training/target에서 별도 absolute-area estimator 호출이 없는지 검사한다.

### Minimal V0 scope 영향

별도 area calibration network/estimator와 ablation을 만들지 않으므로 스코프가 축소된 상태다.

## FV-06 - Shared numerical-rank semantics 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-01 및 V0-04 유지
- **Equation label:** `eq:minimal-tangent-numerical-rank`
- **Source line:** `803--826`

### 원문 LaTeX 전사

```latex
\(J\in\mathbb R^{3\times2}\)의 singular value를
\(\sigma_1(J)\geq\sigma_2(J)\geq0\),
\(G\in\mathbb R^{2\times2}\)의 eigenvalue를
\(\lambda_1(G)\geq\lambda_2(G)\geq0\)로 정렬하고, shared numerical-rank contract를
\begin{equation}
\begin{aligned}
  \sigma_2(J)
  &\geq\max\!\left(\sigma_{J,\mathrm{abs}},\tau_J\sigma_1(J)\right),
  &\sigma_{J,\mathrm{abs}}&>0,\quad 0<\tau_J<1,\\
  \lambda_2(G)
  &\geq\max\!\left(\lambda_{G,\mathrm{abs}}^{(\mathsf u)},
                    \tau_G\lambda_1(G)\right),
  &\lambda_{G,\mathrm{abs}}^{(\mathsf u)}&>0,\quad 0<\tau_G<1
\end{aligned}
  \label{eq:minimal-tangent-numerical-rank}
\end{equation}
로 고정한다. \(\sigma_{J,\mathrm{abs}}\)는 dimensionless이고,
\(\lambda_{G,\mathrm{abs}}^{(\mathsf u)}\)는 SI에서 \(L^2\), nondimensional branch에서
dimensionless인 typed floor이며 SI-to-nd 변환은
\(\widehat\lambda_{G,\mathrm{abs}}=\lambda_{G,\mathrm{abs}}/L_0^2\)다.
각 consumer의 condition cap과 이 threshold를 함께 동결한다. 모든 \(G^{-1}\)은 \(G\)가 이 식과
condition cap을 통과한 뒤에만 계산하고, \(((J^0)^\top J^0)^{-1}\), normal과 area consumer는
해당 \(J\)가 같은 식을 통과한 뒤에만 계산한다. Threshold, unit branch와 CPU/GPU replay 결과를
\(\texttt{tangent\_fit\_v1}\) package hash에 저장한다.
```

### 반례 또는 차원 분석

예를 들어

\[
J=\begin{bmatrix}1&0\\0&10^{-14}\\0&0\end{bmatrix}
\]

는 수학적으로 rank 2지만 simulation/transport 계산에서는 사실상 collapsed support다. 현재 contract는

\[
\sigma_2(J)\ge \max(\sigma_{J,\mathrm{abs}},\tau_J\sigma_1(J))
\]

및 \(G\)의 absolute/relative eigenvalue floor를 공통으로 적용하고 inverse 계산 전에 검사하도록 명시한다. 따라서 플랫폼별 ``rank=2'' tolerance ambiguity가 제거된다.

### 판정 및 Codex action

현재 threshold contract를 surface, bending frame과 Gaussian transport에서 공통 재사용한다.

### Acceptance test

- [ ] Near-rank-1 synthetic J/G가 inverse 전에 fail-fast하는지 검사한다.
- [ ] CPU/GPU 및 save-load replay에서 동일 accept/reject 결과를 확인한다.
- [ ] Rank failure를 arbitrary frame fallback으로 우회하지 않는다.

### Minimal V0 scope 영향

기존 SVD/eigendecomposition threshold만 재사용하며 새 fallback은 없다.

## FV-07 - Layer-label positive-class polarity 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-R1 유지
- **Equation label:** `eq:minimal-topology-loss (label polarity is immediately above); decoder contract`
- **Source line:** `1483--1495; 1497--1522`

### 원문 LaTeX 전사

```latex
Binary label polarity는
\[
\begin{aligned}
  y_{ik}^{\mathrm{rel}}=1
  &\iff \{i,k\}\text{가 valid same-layer material relation이고},\\
  y_{ik}^{\mathrm{layer}}=1
  &\iff \{i,k\}\text{가 invalid cross-layer 또는 geometric shortcut candidate다}
\end{aligned}
\]
로 고정한다. 따라서 valid relation은 \((y_{ik}^{\mathrm{rel}},y_{ik}^{\mathrm{layer}})=(1,0)\)이고,
invalid layer/shortcut은 relation positive로 중복 표기하지 않는다.
\(p_{ik}^{\mathrm{rel}}\)과 \(p_{ik}^{\mathrm{layer}}\)는 각각 이 positive class의 probability이며,
label enum/string, model-output metadata와 decoder metadata를 \(\nu_{\mathrm{label}}\) hash에 함께 저장한다.

Class weight가 동결된 weighted BCE를 \(\operatorname{BCE}_{w}\)라 하면
\begin{equation}
\begin{aligned}
  \mathcal L_{\mathrm{edge}}
  &=|E_{\mathrm{cand}}|^{-1}\sum_{\{i,k\}}\operatorname{BCE}_{w_e}
    (p_{ik}^{\mathrm{rel}},y_{ik}^{\mathrm{rel}}),\\
  \mathcal L_{\mathrm{layer}}
  &=|E_{\mathrm{cand}}|^{-1}\sum_{\{i,k\}}\operatorname{BCE}_{w_l}
    (p_{ik}^{\mathrm{layer}},y_{ik}^{\mathrm{layer}}),\\
  \mathcal L_{\mathrm{area}}
  &=|V|^{-1}\sum_i\left[\log(a_i^0/A_{\mathrm{ref}}+\varepsilon_{a,\log})-
    \log(a_i^\star/A_{\mathrm{ref}}+\varepsilon_{a,\log})\right]^2,\\
  \mathcal L_{\mathrm{topo}}
  &=\lambda_e\mathcal L_{\mathrm{edge}}
    +\lambda_a\mathcal L_{\mathrm{area}}
    +\lambda_{\mathrm{layer}}\mathcal L_{\mathrm{layer}}.
\end{aligned}
  \label{eq:minimal-topology-loss}
\end{equation}
여기서 dimensionless \(\varepsilon_{a,\log}>0\)를 사용하고 teacher area target도
합이 \(A_{\mathrm{ref}}\)가 되도록 정규화한다.
Loss weight, 두 dimensionless epsilon, optimizer/seed와 stopping rule은 \(\nu_{\mathrm{loss}}\)에 저장한다.
Frozen decoder는 \(p_{ik}^{\mathrm{rel}}\geq\tau_{\mathrm{rel}}\),
\(p_{ik}^{\mathrm{layer}}\leq\tau_{\mathrm{layer}}\), degree/radius/layer hard check와 stable-ID tie-break를
모두 통과한 undirected edge만 accept한다. Degree cap 충돌은 score와 stable ID로 결정하고 hidden graph
repair나 per-asset threshold 변경을 금지한다.
```

### 반례 또는 차원 분석

Decoder는 \(p_{ik}^{\mathrm{layer}}\le\tau_{\mathrm{layer}}\)일 때만 edge를 accept한다. 만약 positive class가 ``valid same-layer''였다면 BCE는 좋은 edge에서 \(p_{ik}^{\mathrm{layer}}\to1\)을 학습하고 decoder가 이를 제거하는 모순이 생긴다. 현재 source는

\[
y_{ik}^{\mathrm{layer}}=1
\iff
\{i,k\}\text{가 invalid cross-layer 또는 geometric shortcut candidate}
\]

로 명시하므로 loss와 decoder polarity가 일치한다.

### 판정 및 Codex action

Label enum/string과 decoder metadata를 현재 polarity로 유지한다.

### Acceptance test

- [ ] Valid relation synthetic sample이 (y_rel,y_layer)=(1,0)인지 확인한다.
- [ ] Invalid shortcut sample이 y_layer=1이고 decoder에서 reject되는지 확인한다.
- [ ] Model output metadata와 label generator enum이 동일 hash를 갖는지 확인한다.

### Minimal V0 scope 영향

Network/loss/decoder 구조를 늘리지 않는다.

## FV-08 - Affine-safe SVD, reflection 및 scale clamp 반영 확인

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-04 유지
- **Equation label:** `eq:minimal-affine-safe-map`
- **Source line:** `1269--1316`

### 원문 LaTeX 전사

```latex
\begin{equation}
\begin{aligned}
  F_{j,\mathrm{shell}}^t
  &=U_{F,j}^t
    \operatorname{diag}(\sigma_{F,j1}^t,\sigma_{F,j2}^t,\sigma_{F,j3}^t)
    (V_{F,j}^t)^\top,\\
  \sigma_{F,j1}^t&\geq\sigma_{F,j2}^t\geq\sigma_{F,j3}^t
    \geq\sigma_{F,\mathrm{rank}}>0,
  \qquad \det(F_{j,\mathrm{shell}}^t)>0,\\
  \bar\sigma_{F,jk}^t
  &=\operatorname{clip}(\sigma_{F,jk}^t;\sigma_{F,\min},\sigma_{F,\max}),
  \qquad 0<\sigma_{F,\min}\leq1\leq\sigma_{F,\max},\\
  F_{j,\mathrm{safe}}^t
  &=U_{F,j}^t
    \operatorname{diag}(\bar\sigma_{F,j1}^t,
                        \bar\sigma_{F,j2}^t,
                        \bar\sigma_{F,j3}^t)
    (V_{F,j}^t)^\top .
\end{aligned}
  \label{eq:minimal-affine-safe-map}
\end{equation}
여기서 \(\sigma_{F,\mathrm{rank}},\sigma_{F,\min},\sigma_{F,\max}\)는 dimensionless다.
Non-finite value, numerical rank loss 또는 orientation disagreement는
defensive affine-invalid event다. 다음 orientation failure도 같은 event로 처리한다.
\[
  \det(F_{j,\mathrm{shell}}^t)\leq0.
\]
Singular-vector sign을 바꾸어
reflection을 조용히 proper map으로 수선하지 않고, 식~\eqref{eq:minimal-renderer-rigid-fallback}의 기존
proper-rigid fallback을 시도한다. Affine path가 위 식을 통과하면 그 \(F_{j,\mathrm{safe}}^t\)를 사용하고,
affine path는 실패했지만 rigid path가 성공하면
\(F_{j,\mathrm{safe}}^t=R_{j,\mathrm{rigid}}^t\)로 둔다.

Binding package는 \texttt{tangent\_fit\_version}, ordered-neighbor/weight version,
\(\kappa_{\mathrm{MLS}}\), \(\kappa_{J,\mathrm{MLS}}\),
normal floor, rigid-fallback ID와
\(\tau_K,\tau_{K,\mathrm{ratio}},\tau_{K,\mathrm{plane}},\tau_{K,\mathrm{res}}\),
\(\sigma_{F,\mathrm{rank}},\sigma_{F,\min},\sigma_{F,\max}\), deterministic 3D-SVD ID,
covariance eigenvalue bound와
development에서 동결한 renderer fallback-rate cap \(\tau_{\mathrm{rigid}}\)을 저장한다.
Required affine attempt 중 invalid와 singular-value clip event 비율
\(r_{\mathrm{affine,invalid}},r_{\mathrm{affine,clip}}\)을 각각 trace에 기록하고,
event definition, threshold와 package hash를 함께 저장한다.
Run의 required Gaussian--frame transport attempt 수에 대한 successful rigid-fallback event 비율을
\(r_{\mathrm{rigid}}\)로 기록한다. Rigid fallback이 성공하고
\(r_{\mathrm{rigid}}\leq\tau_{\mathrm{rigid}}\)이면 fallback-marked renderer 결과를 사용할 수 있다.
Rigid fit 자체가 실패하거나 cap을 넘으면 식~\eqref{eq:minimal-attempt-success-contract}의
in-envelope method failure다. 이 fallback은 식~\eqref{eq:minimal-surface-tangent-fit}의
```

### 반례 또는 차원 분석

예를 들어 singular values가 \((100,1,0.01)\)인 affine map을 그대로 covariance에 적용하면 principal covariance scale이 대략 \((10^4,1,10^{-4})\)배로 변한다. 현재 source는

\[
\bar\sigma_{F,jk}=\operatorname{clip}(\sigma_{F,jk};\sigma_{F,\min},\sigma_{F,\max})
\]

을 mean/covariance 공통 map에 적용하고, \(\det(F_{j,\mathrm{shell}}^t)\le0\)이면 singular-vector sign으로 조용히 수선하지 않고 기존 proper-Kabsch fallback으로 보낸다. Identity와 proper rigid motion에서는 모든 singular value가 1이므로 map이 바뀌지 않는다.

### 판정 및 Codex action

현재 affine-safe equation과 renderer-only fallback을 유지한다.

### Acceptance test

- [ ] Identity와 random proper rigid motion에서 F_safe가 각각 I와 Q를 재현한다.
- [ ] Extreme singular values가 frozen bounds로 clip되는지 검사한다.
- [ ] Reflection candidate가 affine-invalid로 기록되고 proper-rigid fallback으로만 이동하는지 검사한다.
- [ ] r_affine_invalid, r_affine_clip, r_rigid가 trace와 hash에 기록되는지 검사한다.

### Minimal V0 scope 영향

기존 3x3 SVD와 rigid fallback을 재사용하며 새 transport stage가 없다.

## FV-09 - Dimensionless log-PSD metric 반영 확인

- **PRIORITY:** `KEEP FORMULA / PATCH NOTATION IN FV-13`
- **판정:** **수용**
- **적용 시점:** 평가 스크립트 유지
- **Equation label:** `eq:minimal-log-psd-errors`
- **Source line:** `1686--1715`

### 원문 LaTeX 전사

```latex
를 계산한다. 이 절의 \(f\)는 force가 아니라 temporal frequency다.
PSD는 per-bin power가 아니라 one-sided velocity-density PSD convention으로 고정하며
\([P]=L^2/T\)다. Sampling rate, window normalization, DC/Nyquist 처리와 development에서 동결한
frequency bin \(B\)를 evaluation hash에 저장한다. PSD와 같은 unit을 가진
\(\varepsilon_P>0\)를 사용하고, unit-parity를 나타내는 기존 typed conversion scale
\((L_0,T_0)\)에 대해
\begin{equation}
\begin{aligned}
  \widehat f&=fT_0,&
  \widehat\varepsilon_P&=\varepsilon_P\frac{T_0}{L_0^2},\\
  \widehat P_s(\widehat f)
  &=P_s(f)\frac{T_0}{L_0^2},\\
  \widehat P_{\mathrm{ref},s}(\widehat f)
  &=P_{\mathrm{ref},s}(f)\frac{T_0}{L_0^2},\\
  \delta_s(\widehat f)
  &=\log\!\left(\widehat P_s(\widehat f)+\widehat\varepsilon_P\right)
    -\log\!\left(\widehat P_{\mathrm{ref},s}(\widehat f)+\widehat\varepsilon_P\right),\\
  e_{\mathrm{PSD,RMS},s}
  &=\left(|\widehat B|^{-1}\sum_{\widehat f\in\widehat B}
    \delta_s(\widehat f)^2\right)^{1/2},\\
  e_{\mathrm{PSD,L1},s}
  &=|\widehat B|^{-1}\sum_{\widehat f\in\widehat B}|\delta_s(\widehat f)|.
\end{aligned}
  \label{eq:minimal-log-psd-errors}
\end{equation}
를 둘 다 보고한다. 여기서 \(\widehat B=\{fT_0:f\in B\}\)다. 같은 값은
\(\delta_s(f)=\log[(P_s(f)+\varepsilon_P)/(P_{\mathrm{ref},s}(f)+\varepsilon_P)]\)로 직접
계산할 수 있으므로 SI direct branch에 새 독립 \(T_{\mathrm{PSD,ref}}\)를 만들지 않는다.
\(\varepsilon_P\)의 typed value/conversion과 위 convention 전체를 evaluation hash에 포함한다.
Signed mean log-PSD처럼 cancellation이 가능한 값 하나로 대체하지 않는다.
```

### 반례 또는 차원 분석

One-sided velocity-density PSD는 \([P]=L^2/T\)다. 현재 식의

\[
\widehat P=P\frac{T_0}{L_0^2},\qquad
\widehat\varepsilon_P=\varepsilon_P\frac{T_0}{L_0^2}
\]

는 둘 다 무차원이며, log ratio는 unit change에 불변이다. 따라서 이전처럼 dimensional quantity를 직접 로그에 넣는 문제는 해결되었다. 남은 문제는 force와 temporal frequency가 모두 \(f,\widehat f\)를 사용하는 표기 충돌뿐이며 이는 FV-12에서 명칭만 수정한다.

### 판정 및 Codex action

PSD normalization과 metric은 유지하고 frequency symbol만 FV-12에 따라 변경한다.

### Acceptance test

- [ ] m/s와 cm/s 단위 표현에서 dimensionless PSD error가 동일한지 검사한다.
- [ ] Low-power bin에서 epsilon 변환이 PSD와 동일 scale을 따르는지 검사한다.
- [ ] Window/DC/Nyquist/bin convention이 evaluation hash에 포함되는지 검사한다.

### Minimal V0 scope 영향

Runtime physics에는 영향이 없고 evaluation variable rename만 남는다.

## FV-10 - V0-R1 canonical config freeze lifecycle 확인

- **PRIORITY:** `FOLLOW-UP HOLD / BLOCK TRAINING UNTIL SET`
- **판정:** **후속 보류**
- **적용 시점:** V0-R1 training entry 직전
- **Equation label:** `eq:minimal-r1-version-contract; eq:minimal-r1-anchor-candidate`
- **Source line:** `1411--1432; 1524--1541`

### 원문 LaTeX 전사

```latex
Topology distillation은 runtime network가 아니라 setup용 V0-R1 module 하나다. 그 실행 계약을
\begin{equation}
  \mathcal V_{\mathrm{R1}}
  =(\nu_{\mathrm{anchor}},\nu_{\mathrm{cand}},\nu_{\mathrm{feat}},
    \nu_{\mathrm{arch}},\nu_{\mathrm{label}},\nu_{\mathrm{loss}},
    \nu_{\mathrm{decode}},\nu_{\mathrm{split}})
  \label{eq:minimal-r1-version-contract}
\end{equation}
로 저장한다. 각 \(\nu\)는 algorithm ID, normalization, threshold/cap, seed와 deterministic tie-break를
포함한다. Surface-valid Gaussian에서 anchor를 만드는
\(\mathcal S_{\mathrm{anchor}}^{\nu_{\mathrm{anchor}}}\)와 각 anchor의 broad proposal set
\(\mathcal P_i^{\nu_{\mathrm{cand}}}\)를 config에서 정확히 하나 선택하고
\begin{equation}
  V=\mathcal S_{\mathrm{anchor}}^{\nu_{\mathrm{anchor}}}(\mathcal G_{\mathrm{valid}}),
  \qquad
  E_{\mathrm{cand}}=\bigl\{\{i,k\}:k\in\mathcal P_i\ \text{or}\ i\in\mathcal P_k\bigr\}
  \label{eq:minimal-r1-anchor-candidate}
\end{equation}
로 undirected candidate graph를 만든다. Weighted FPS, deterministic radius/kNN 또는 이들의 고정
조합은 허용 가능한 config 후보지만 어느 하나를 방법 전체의 유일한 선택으로 선언하지 않는다.
각 training run은 아래 lifecycle에 따라 routine, cardinality/radius와 normalization이 모두 채워진
정확히 한 training-static config만 사용하며, asset별 retuning을 금지한다.

Split의 독립 단위는 source object group이다. 같은 source object의 GS density/seed/reconstruction variant는
한 group에만 속하고 train/development/test 사이에 나뉘지 않는다. V0-R1 lifecycle은 다음 순서만 허용한다.
\begin{enumerate}
  \item 먼저 \(\nu_{\mathrm{split}}\), object-group assignment와 data hash를 동결하고 test object/package ID를
  seal한다. 이후 config/model/decoder 선택 동안 test payload와 result를 열지 않는다.
  \item 각 training entry 전에
  \(\nu_{\mathrm{anchor}},\nu_{\mathrm{cand}},\nu_{\mathrm{feat}},\nu_{\mathrm{arch}},
  \nu_{\mathrm{label}},\nu_{\mathrm{loss}},\nu_{\mathrm{split}}\), decoder schema, seed와 training data/label
  hash가 non-placeholder인 training-static config인지 검사하고 하나라도 비었으면 fail-fast한다.
  \item Preregistered candidate run은 train group만 학습하고 development group만으로 정확히 한
  model/config와 decoder threshold/cap을 선택한다. Asset별 또는 test-aware retuning은 금지한다.
  \item Test payload를 처음 열기 전에 \(\mathcal V_{\mathrm{R1}}\)의 모든 field와 선택한 model weight를
  동결한다. Training source, label, data, config, decoder와 split의 full hash도 같은 immutable
  manifest에 기록한다.
  Test failure 뒤 어느 field라도 바꾸면 새 preregistered run이지 같은 result의 보정이 아니다.
\end{enumerate}
이 lifecycle의 미완료나 실패는 V0-R1 predicted scaffold와 Gate A/MC1 evidence만 막는다.
Explicit Oracle scaffold의 V0-01--04, Oracle Gate B와 dynamics-only Gate C 경로를 막지 않는다.
```

### 반례 또는 차원 분석

동일 GS에서도 fixed kNN, fixed radius, radius-kNN union은 candidate graph와 package acceptance를 다르게 만든다. 예를 들어 sparse region에서는 radius graph가 isolated node를 만들 수 있고 kNN은 지나치게 긴 edge를 만들 수 있다. 따라서 실제 \(\nu_{\mathrm{anchor}},\nu_{\mathrm{cand}},\nu_{\mathrm{label}},\nu_{\mathrm{decode}}\) 값이 없는 상태로 predicted-scaffold training/result를 시작할 수는 없다. 현재 source는 non-placeholder check, development-only selection, test-open 이전 full freeze를 명시하므로 lifecycle은 올바르다.

### 판정 및 Codex action

V0-01--04 Oracle scaffold 경로는 진행하되, V0-R1 training entry에서 실제 config 값이 비어 있으면 fail-fast한다. 모든 후보를 구현하지 말고 preregistered development 후보 중 정확히 하나만 동결한다.

### Acceptance test

- [ ] Training entry에서 모든 nu field와 seed/data/label hash의 non-placeholder 여부를 검사한다.
- [ ] Test payload open 이전 model/config/decoder full hash가 immutable인지 검사한다.
- [ ] Test failure 뒤 field 변경은 새 run ID로만 허용한다.

### Minimal V0 scope 영향

새 sampler/graph family를 추가하는 요구가 아니라 이미 정의된 후보 중 하나를 선택·동결하는 작업이다.

## FV-11 - Oracle-selected Local 명칭의 과도한 upper-bound claim

- **PRIORITY:** `PATCH WORDING NOW`
- **판정:** **반박 (명칭만)**
- **적용 시점:** V0-03/Gate B 문서 동결 전
- **Equation label:** `No equation label; Gate B and baseline text`
- **Source line:** `1576--1592; 1627--1633; 1850; 1864`

### 원문 LaTeX 전사

```latex
동일 p95 dynamics latency에서

\[
  \text{strong Global rank sweep}
  \quad\text{vs.}\quad
  \text{Global + Oracle-selected complementary Local}
\]

을 비교한다. Oracle Local이 tip flutter, traveling gust, velocity trajectory 또는
target-band PSD에서 식~\eqref{eq:minimal-pareto-effects}의 clustered effect-margin improvement를
반복적으로 만들지 못하면
MC2와 selector 개발을 중단하고 Global-only 방향으로 축소한다.

Oracle selector는 setup에서 동결한 동일 atomic units와 \(K_A,K_D\), 동일 coupled solver를 사용하되,
privileged teacher의 다음 \(H\) frame velocity/target-band error 감소량으로 unit을 rank한다.
Future label search 비용은 runtime latency에서 제외하지만 선택 뒤 실제 execution 비용은 포함한다.
\(H\), error metric과 tie-break를 development split에서 한 번 동결한다.

\begin{enumerate}
  \item High-resolution prescribed-wind structural reference: accuracy reference 전용
  \item 동일 constraint와 timestep을 쓰는 optimized full-coordinate same-anchor PD backend
  \item Global ROM rank sweep
  \item Global + always-on Local upper bound
  \item Global + Oracle-selected Local
  \item Global + deterministic selected Local

  \item \textbf{V0-03:} Oracle Local, deterministic selector, frozen decay/release schedule,

Gate B는 V0-03의 Oracle Local 결과가 생기면, Gate C의 dynamics-only 판정은 V0-02 baseline과
```

### 반례 또는 차원 분석

현재 teacher는 각 unit의 다음 \(H\)-frame error 감소량으로 개별 ranking한다. 그러나 patch cross block과 mode redundancy 때문에 subset benefit은 일반적으로 additive하지 않다. 예를 들어

\[
\Delta(\{A\})=10,\quad \Delta(\{B\})=9,\quad \Delta(\{C\})=8.5,
\]

이지만

\[
\Delta(\{A,B\})=10.5,\qquad
\Delta(\{A,C\})=18.0
\]

일 수 있다. Per-unit ranking은 \(A,B\)를 고르지만 best subset은 \(A,C\)다. 따라서 \(K_A>1\)에서 현재 방법은 exact combinatorial oracle이 아니며 ``Oracle-selected''라는 표현은 실제 정의보다 강하다.

### 판정 및 Codex action

Selector 코드와 teacher rollout은 유지하고 selector-context의 명칭만 privileged-teacher-ranked로 축소한다. Oracle scaffold라는 표현은 실제 source/explicit scaffold를 뜻하므로 변경하지 않는다.

### 권장 replacement LaTeX

```latex
% Gate B comparison
\[
  \text{strong Global rank sweep}
  \quad\text{vs.}\quad
  \text{Global + privileged-teacher-ranked complementary Local}
\]

Privileged-teacher-ranked Local이 tip flutter, traveling gust, velocity trajectory 또는
target-band PSD에서 식~\eqref{eq:minimal-pareto-effects}의 clustered effect-margin improvement를
반복적으로 만들지 못하면 MC2와 selector 개발을 중단하고 Global-only 방향으로 축소한다.

Privileged teacher selector는 setup에서 동결한 동일 atomic units와 \(K_A,K_D\),
동일 coupled solver를 사용하되, privileged teacher의 다음 \(H\) frame
velocity/target-band error 감소량으로 unit을 rank한다.

% Required baseline item
\item Global + privileged-teacher-ranked Local

% Implementation order and early Gate B text
\item \textbf{V0-03:} privileged-teacher-ranked Local, deterministic selector,
frozen decay/release schedule, single coupled solve와 S5

Gate B는 V0-03의 privileged-teacher-ranked Local 결과가 생기면, ...
```

### Acceptance test

- [ ] Selector-context에서 Oracle-selected, Oracle selector, Oracle Local 문자열이 남지 않는지 검사한다.
- [ ] Oracle scaffold 및 Oracle flat-rest scaffold 문자열은 변경하지 않는다.
- [ ] Teacher가 exact best-subset upper bound라는 claim을 논문/figure caption에서 사용하지 않는다.

### Minimal V0 scope 영향

문구와 baseline label만 변경하며 selector, solver, search budget 및 runtime에는 영향이 없다.

## FV-12 - Area raw logit의 생성식 누락

- **PRIORITY:** `PATCH BEFORE V0-R1`
- **판정:** **수용**
- **적용 시점:** V0-R1 architecture freeze 전
- **Equation label:** `eq:minimal-r1-symmetric-logit; eq:minimal-r1-area-decode`
- **Source line:** `1451--1467`

### 원문 LaTeX 전사

```latex
Node encoder \(h_i=E_\theta(\phi_i)\) 뒤 relation/layer logit은
\begin{equation}
  \ell_{ik}^{c}
  =g_{\theta,c}\!\left(h_i+h_k,|h_i-h_k|,\chi_{ik}\right),\qquad
  p_{ik}^{c}=\sigma(\ell_{ik}^{c})=p_{ki}^{c},\quad
  c\in\{\mathrm{rel},\mathrm{layer}\}
  \label{eq:minimal-r1-symmetric-logit}
\end{equation}
처럼 symmetry를 construction으로 보장한다. Width/depth/activation은
\(\nu_{\mathrm{arch}}\)에서 하나로 동결하되 특정 architecture를 method claim으로 만들지 않는다.
Area raw logit \(\zeta_i^a\)와 dimensionless floor
\(\varepsilon_{a,\mathrm{raw}}>0\)에 대해
\begin{equation}
  \widetilde a_i=\operatorname{softplus}(\zeta_i^a)+\varepsilon_{a,\mathrm{raw}},\qquad
  a_i^0=A_{\mathrm{ref}}\frac{\widetilde a_i}{\sum_{k\in V}\widetilde a_k}
  \label{eq:minimal-r1-area-decode}
\end{equation}
```

### 반례 또는 차원 분석

현재 source만으로는 다음 구현이 모두 가능하다.

\[
\zeta_i^a=g_{\theta,a}(h_i),\qquad
\zeta_i^a=g_{\theta,a}(\phi_i),\qquad
\zeta_i^a=g_{\theta,a}\!\left(h_i,|V|^{-1}\sum_k h_k\right).
\]

세 번째는 global pooling을 추가하므로 현재의 단순 node-encoder contract보다 범위가 넓다. Codex가 architecture를 자의적으로 확장하지 않도록 area head가 node embedding \(h_i\)만 소비한다고 한 줄로 닫아야 한다.

### 판정 및 Codex action

기존 area decoder label을 유지하고 scalar node head를 명시한다. 이는 이미 예정된 area output을 정의하는 것이며 새 module이 아니다.

### 권장 replacement LaTeX

```latex
Area head는 node embedding만 소비한다. Dimensionless floor
\(\varepsilon_{a,\mathrm{raw}}>0\)에 대해
\begin{equation}
\begin{aligned}
  \zeta_i^a
  &=g_{\theta,a}(h_i),\\
  \widetilde a_i
  &=\operatorname{softplus}(\zeta_i^a)+\varepsilon_{a,\mathrm{raw}},\\
  a_i^0
  &=A_{\mathrm{ref}}
    \frac{\widetilde a_i}{\sum_{k\in V}\widetilde a_k}.
\end{aligned}
  \label{eq:minimal-r1-area-decode}
\end{equation}
Area-head width/depth/activation과 output field ID는 \(\nu_{\mathrm{arch}}\)가 소유한다.
```

### Acceptance test

- [ ] Area head input tensor가 h_i뿐이며 global pooling 또는 new message-passing stage가 없는지 검사한다.
- [ ] Area logit output shape이 |V| x 1인지 검사한다.
- [ ] a_i^0 > 0 및 sum_i a_i^0 = A_ref를 tolerance 내 확인한다.
- [ ] Area-head architecture metadata가 nu_arch hash에 포함되는지 검사한다.

### Minimal V0 scope 영향

기존 node encoder 뒤 scalar output head를 명시할 뿐 새 network/module/runtime stage를 만들지 않는다.

## FV-13 - Force와 temporal frequency의 f 표기 충돌

- **PRIORITY:** `PATCH NOTATION BEFORE EVALUATION FREEZE`
- **판정:** **수용**
- **적용 시점:** Gate A/B evaluation code 동결 전
- **Equation label:** `eq:minimal-si-nd-typed-map; eq:minimal-log-psd-errors`
- **Source line:** `195--214; 1686--1715`

### 원문 LaTeX 전사

```latex
\begin{equation}
\begin{aligned}
  \widehat x&=x/L_0,&
  \widehat t&=t/T_0,&
  \widehat{\Delta t}&=\Delta t/T_0,\\
  \widehat m&=m/M_0,&
  \widehat a&=a/L_0^2,&
  \widehat\Sigma&=\Sigma/L_0^2,\\
  \widehat\rho_A&=\rho_A L_0^2/M_0,&
  \widehat\kappa_{\mathrm{str}}&=\kappa_{\mathrm{str}}T_0^2/M_0,&
  \widehat\kappa_{\mathrm{bend}}&=\kappa_{\mathrm{bend}}T_0^2/(M_0L_0^2),\\
  \widehat\rho_{\mathrm{air}}&=\rho_{\mathrm{air}}L_0^3/M_0,&
  \widehat W&=WT_0/L_0,&
  \widehat{\partial_tW}&=(\partial_tW)T_0^2/L_0,\\
  \widehat{\nabla_xW}&=T_0\nabla_xW,&
  \widehat f&=fT_0^2/(M_0L_0),&
  \widehat f_{\max}&=f_{\max}T_0^2/(M_0L_0).
\end{aligned}
  \label{eq:minimal-si-nd-typed-map}
\end{equation}

를 계산한다. 이 절의 \(f\)는 force가 아니라 temporal frequency다.
PSD는 per-bin power가 아니라 one-sided velocity-density PSD convention으로 고정하며
\([P]=L^2/T\)다. Sampling rate, window normalization, DC/Nyquist 처리와 development에서 동결한
frequency bin \(B\)를 evaluation hash에 저장한다. PSD와 같은 unit을 가진
\(\varepsilon_P>0\)를 사용하고, unit-parity를 나타내는 기존 typed conversion scale
\((L_0,T_0)\)에 대해
\begin{equation}
\begin{aligned}
  \widehat f&=fT_0,&
  \widehat\varepsilon_P&=\varepsilon_P\frac{T_0}{L_0^2},\\
  \widehat P_s(\widehat f)
  &=P_s(f)\frac{T_0}{L_0^2},\\
  \widehat P_{\mathrm{ref},s}(\widehat f)
  &=P_{\mathrm{ref},s}(f)\frac{T_0}{L_0^2},\\
  \delta_s(\widehat f)
  &=\log\!\left(\widehat P_s(\widehat f)+\widehat\varepsilon_P\right)
    -\log\!\left(\widehat P_{\mathrm{ref},s}(\widehat f)+\widehat\varepsilon_P\right),\\
  e_{\mathrm{PSD,RMS},s}
  &=\left(|\widehat B|^{-1}\sum_{\widehat f\in\widehat B}
    \delta_s(\widehat f)^2\right)^{1/2},\\
  e_{\mathrm{PSD,L1},s}
  &=|\widehat B|^{-1}\sum_{\widehat f\in\widehat B}|\delta_s(\widehat f)|.
\end{aligned}
  \label{eq:minimal-log-psd-errors}
\end{equation}
를 둘 다 보고한다. 여기서 \(\widehat B=\{fT_0:f\in B\}\)다. 같은 값은
\(\delta_s(f)=\log[(P_s(f)+\varepsilon_P)/(P_{\mathrm{ref},s}(f)+\varepsilon_P)]\)로 직접
계산할 수 있으므로 SI direct branch에 새 독립 \(T_{\mathrm{PSD,ref}}\)를 만들지 않는다.
\(\varepsilon_P\)의 typed value/conversion과 위 convention 전체를 evaluation hash에 포함한다.
Signed mean log-PSD처럼 cancellation이 가능한 값 하나로 대체하지 않는다.
```

### 반례 또는 차원 분석

첫 번째 \(f\)는 force이므로 \([f]=ML/T^2\)이고

\[
\widehat f=f\frac{T_0^2}{M_0L_0}
\]

가 맞다. 두 번째 \(f\)는 temporal frequency이므로 \([f]=T^{-1}\)이고 \(\widehat f=fT_0\)가 맞다. 두 식은 각각 차원상 올바르지만 동일 symbol이 서로 다른 typed quantity를 나타내므로 Codex가 unit conversion helper 또는 evaluation variable을 잘못 공유할 위험이 있다.

### 판정 및 Codex action

Force symbol은 유지하고 PSD 절의 temporal frequency만 \(\nu,\widehat\nu\)로 변경한다. Equation label과 metric 정의는 유지한다.

### 권장 replacement LaTeX

```latex
이 절의 \(\nu\)는 temporal frequency다. PSD는 per-bin power가 아니라
one-sided velocity-density PSD convention으로 고정하며 \([P]=L^2/T\)다.
\begin{equation}
\begin{aligned}
  \widehat\nu&=\nu T_0,&
  \widehat\varepsilon_P&=\varepsilon_P\frac{T_0}{L_0^2},\\
  \widehat P_s(\widehat\nu)
  &=P_s(\nu)\frac{T_0}{L_0^2},\\
  \widehat P_{\mathrm{ref},s}(\widehat\nu)
  &=P_{\mathrm{ref},s}(\nu)\frac{T_0}{L_0^2},\\
  \delta_s(\widehat\nu)
  &=\log\!\left(\widehat P_s(\widehat\nu)+\widehat\varepsilon_P\right)
    -\log\!\left(\widehat P_{\mathrm{ref},s}(\widehat\nu)+\widehat\varepsilon_P\right),\\
  e_{\mathrm{PSD,RMS},s}
  &=\left(|\widehat B|^{-1}\sum_{\widehat\nu\in\widehat B}
    \delta_s(\widehat\nu)^2\right)^{1/2},\\
  e_{\mathrm{PSD,L1},s}
  &=|\widehat B|^{-1}\sum_{\widehat\nu\in\widehat B}
    |\delta_s(\widehat\nu)|.
\end{aligned}
  \label{eq:minimal-log-psd-errors}
\end{equation}
여기서 \(\widehat B=\{\nu T_0:\nu\in B\}\)다. 같은 값은
\(\delta_s(\nu)=\log[(P_s(\nu)+\varepsilon_P)/(P_{\mathrm{ref},s}(\nu)+\varepsilon_P)]\)로
직접 계산할 수 있다.
```

### Acceptance test

- [ ] eq:minimal-si-nd-typed-map의 force mapping은 변경하지 않는다.
- [ ] PSD section에 temporal-frequency assignment가 f 계열로 남지 않는지 검사한다.
- [ ] Evaluation code의 frequency variable과 hash field가 nu 계열로 일치하는지 검사한다.
- [ ] 기존 dimensionless PSD parity test 결과가 변경되지 않는지 확인한다.

### Minimal V0 scope 영향

표기와 evaluation variable name만 바뀌며 metric, runtime physics, baseline에는 영향이 없다.

## 4. 기계적 소스 검사

- Source lines: `1915`
- Labels: `47` total, `47` unique, duplicate `[]`
- `eqref/ref` missing labels: `[]`
- Environment `equation` begin/end: `48/48`
- Environment `aligned` begin/end: `27/27`
- Environment `enumerate` begin/end: `9/9`
- Environment `itemize` begin/end: `7/7`
- Environment `longtable` begin/end: `4/4`
- Environment `tcolorbox` begin/end: `2/2`
- Environment `gathered` begin/end: `1/1`
- XeLaTeX two-pass compile: fatal syntax error 없음. 제공된 `.bib`가 현재 입력 폴더에 없어 citation warning은 남는다.

## 5. 최종 Do-Not-Expand 명세

- Bending projector, fixed-Hessian package modes, one-frame staggered aero, direct dense reduced solve와 current selector는 유지한다.
- Exact best-subset oracle search, tangent-consistent basis, new area estimator, learned gate, same-frame corrector 또는 new shell mechanics를 추가하지 않는다.
- Gate B에서 teacher-ranked Local이 strong Global rank sweep을 이기지 못하면 selector를 확장하지 말고 MC2를 축소한다.
- Gate C에서 observed p95 bottleneck이 확인되기 전 advanced solver를 추가하지 않는다.

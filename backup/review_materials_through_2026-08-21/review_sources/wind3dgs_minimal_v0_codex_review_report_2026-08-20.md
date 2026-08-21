---
title: "Minimal V0 수정본 리뷰 및 Codex 패치 명세"
document_type: "codex_review_patch_spec"
date: "2026-08-20"
source_file: "붙여넣은 텍스트 (1).txt"
source_sha256: "0408329ea6c50584164cd752e45eb103ba00f85789aa90a51bbf1c08439cabd8"
scope_policy: "No new solver/module unless Gate failure or observed visual artifact requires it"
---

# Minimal V0 수정본 리뷰 및 Codex 패치 명세

## 0. 문서 역할
이 파일은 리뷰 결과를 사람이 읽을 수 있는 보고서이자 VS Code Codex extension에 직접 입력할 수 있는 패치 명세로 구성한다. Source line은 위 SHA-256의 정확한 입력 파일을 기준으로 한다. 다른 버전에 적용할 때는 equation label을 우선 사용하고, line number는 보조 기준으로만 사용한다.

### 절대 스코프 규칙
- 새 solver, shell backend, runtime network, corrector, KKT/Schur, hyper-reduction 또는 추가 mainline을 만들지 않는다.
- `KEEP / NO PATCH`, `감수`, `PAPER-STAGE ONLY` 항목은 명시된 문구 외 코드를 변경하지 않는다.
- 기존 equation label은 가능한 한 유지한다. 새 label은 `RV-08`, `RV-10`에서 명시한 contract equation에만 허용한다.
- 관련 없는 formatting, 변수명, 파일 구조를 정리하지 않는다.
- patch 뒤 canonical source를 XeLaTeX로 compile하고 아래 fixture를 실행한다.

## 1. 종합 판정
수정본은 이전 P0 bending 오류를 해결했고 SI/nd 및 release 차원 계약도 올바르게 닫았다. 현재 새 solver/module을 요구하는 치명적 오류는 없다. Codex가 실제로 변경해야 하는 것은 contract/metric/transport 정의의 제한된 패치뿐이다.

| 분류 | Issue | 적용 시점 | 결과 |
|---|---|---|---|
| KEEP / NO PATCH | RV-01 Corotated tangent-plane bending 수정의 수학적 타당성 | V0-01 유지 | 수용 |
| PATCH NOW | RV-02 Attachment-line eigenvector의 unit normalization 누락 | V0-01 이전 | 수용 |
| KEEP / NO PATCH | RV-03 SI--nondimensional typed map과 release proxy의 차원 일관성 | V0-00 및 V0-03 유지 | 수용 |
| PATCH WORDING ONLY | RV-04 Fixed-Hessian K와 minimized bending energy의 exact tangent 불일치 | V0-01 문구 정정 | 감수 |
| PATCH BEFORE V0-R1 | RV-05 A_ref area estimator가 불필요한 calibration scope를 만드는 문제 | V0-R1 이전 | 수용 |
| DEFERRED DECISION / BLOCK TRAINING UNTIL SET | RV-06 V0-R1 version contract는 있으나 canonical config 값은 아직 미동결 | V0-R1 학습 시작 직전 | 후속 보류 |
| PATCH BEFORE V0-R1 | RV-07 Layer label의 positive-class polarity 미정의 | V0-R1 label generator 구현 전 | 수용 |
| PATCH BEFORE V0-04 | RV-08 F_safe의 reflection 및 singular-value clamp 규칙 미완전 정의 | V0-04 transport 구현 전 | 수용 |
| PATCH BEFORE EVALUATION FREEZE | RV-09 Log-PSD metric에 dimensional PSD를 직접 입력 | Gate A/B 평가 스크립트 동결 전 | 수용 |
| PATCH NOW | RV-10 Surface/transport tangent의 rank=2가 numerical rank로 정의되지 않음 | V0-01 tangent fit 구현 전 | 수용 |
| PAPER-STAGE ONLY | RV-11 Canonical implementation specification과 논문 본문의 exposition 분리 | V0-05 이후 논문 원고 작성 | 감수 |

### Codex 적용 순서
1. V0-01 이전: `RV-02`, `RV-10`, `RV-04`의 용어 패치.
2. V0-04 이전: `RV-08`.
3. V0-R1 이전: `RV-05`, `RV-06`, `RV-07`.
4. 평가 스크립트 동결 전: `RV-09`.
5. 변경 금지/유지 확인: `RV-01`, `RV-03`.
6. V0-05 이후 paper draft에서만: `RV-11`.

## 2. 상세 리뷰

## RV-01 - Corotated tangent-plane bending 수정의 수학적 타당성

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-01 유지
- **Equation label:** `eq:minimal-bending-corotated-frame`, `eq:minimal-bending-corotated-projector`
- **Source line:** 476--511

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
Rest frame을 고정하고 \(n=n_{i,\mathrm{fit}}^0\),
\(b=((n)^\top d_{ia}^0)n\)로 둔다. Normal perturbation
\(d(\epsilon)=d_{ia}^0+\epsilon n\)에 대해
\[
  n^\top(d_{ia}^0-b)=0,
  \qquad
  \Pi(d(\epsilon))=d_{ia}^0,
\]
이므로 residual과 energy는
\[
  d(\epsilon)-\Pi(d(\epsilon))=\epsilon n,
  \qquad
  E(\epsilon)=\frac12 w_{ia}^{\mathrm{bend}}\epsilon^2.
\]
따라서
\[
  \left.\frac{d^2E}{d\epsilon^2}\right|_{\epsilon=0}
  =w_{ia}^{\mathrm{bend}}>0.
\]
기존 nonzero-\(d_{ia}^0\) sphere orbit의 \(O(\epsilon^4)\) 퇴화가 제거된다.
Proper rigid transform \(Q\in SO(3)\)에서도
\(d'=Qd_{ia}^0\), \(n'=Qn\), \(b'=Qb\)이므로
\(\Pi'(Qd_{ia}^0)=Qd_{ia}^0\)가 성립한다.

### 판정 및 Codex action
수식과 projector를 변경하지 않는다. 현재 unit/integrated fixture를 유지한다.

### Acceptance test
- [ ] Rest exactness: Q_i^0=I 및 projector residual 0.
- [ ] Frozen-frame central finite difference: normal stiffness가 w_bend와 허용오차 내 일치.
- [ ] Random proper rigid transform reproduction.
- [ ] Integrated perturbation에서 positive non-collapsed normal response.

### Minimal V0 scope 영향
새 constraint, solver, local/global iteration 또는 runtime stage가 없다. 기존 projector의 정정이 완료된 상태다.

## RV-02 - Attachment-line eigenvector의 unit normalization 누락

- **PRIORITY:** `PATCH NOW`
- **판정:** **수용**
- **적용 시점:** V0-01 이전
- **Equation label:** `eq:minimal-attachment-line-gauge`
- **Source line:** 384--390, 395--402

### 원문 LaTeX 전사
```latex
\begin{equation}
  C_{\mathrm{att}}g_{\mathrm{att}}=\lambda_1g_{\mathrm{att}},\qquad
  L_{\mathrm{att}}=g_{\mathrm{att}}g_{\mathrm{att}}^\top,\qquad
  \frac{\lambda_1-\lambda_2}{\lambda_1+\varepsilon_{\mathrm{att}}}
  \geq\tau_{\mathrm{gap}}>0
  \label{eq:minimal-attachment-line-gauge}
\end{equation}
를 요구한다. 물리적 gauge는 vector \(g_{\mathrm{att}}\)가 아니라 sign-invariant unoriented line
\(L_{\mathrm{att}}\)다. \(\lambda_1\) floor나 eigengap을 통과하지 못하면 world-axis에 맞춘 임의 sign 또는
secondary direction으로 우회하지 않고 package를 reject한다.

Anchor \(i\)에서는 \(P_i^0=I-n_i^0(n_i^0)^\top\),
\(q_i=P_i^0g_{\mathrm{att}}\)에 대해 \(\|q_i\|\geq\tau_{\mathrm{gauge}}\)를 요구하고
\begin{equation}
  L_i^0=\frac{q_iq_i^\top}{q_i^\top q_i},\qquad
  t_{i1}^0\in\operatorname{range}(L_i^0),\quad \|t_{i1}^0\|_2=1,qquad
  t_{i2}^0=n_i^0\times t_{i1}^0
  \label{eq:minimal-local-line-gauge}
\end{equation}
```

### 반례 또는 차원 분석
Eigenvector는 scale에 대해 유일하지 않다. \(g_{\mathrm{att}}\)가 해이면
임의의 \(c\neq0\)에 대해 \(cg_{\mathrm{att}}\)도 해다. 현재 식에서는
\[
  L_{\mathrm{att}}'=c^2L_{\mathrm{att}},
  \qquad
  \|P_i^0(cg_{\mathrm{att}})\|_2
  =|c|\,\|P_i^0g_{\mathrm{att}}\|_2.
\]
따라서 동일 geometry라도 eigensolver가 반환한 eigenvector scale에 따라
\(\|q_i\|\ge\tau_{\mathrm{gauge}}\) acceptance가 달라질 수 있다.
예를 들어 원래 projected norm이 0.2, threshold가 0.1인데
\(c=0.1\)이면 0.02가 되어 같은 package가 reject된다.

### 판정 및 Codex action
기존 equation label을 유지하고 \(\|g_{\mathrm{att}}\|_2=1\)을 추가한다. 구현에서는 eigensolver 출력 직후 normalize한 뒤 q_i와 L_att를 계산한다.

### 권장 replacement LaTeX
```latex
\begin{equation}
  C_{\mathrm{att}}g_{\mathrm{att}}=\lambda_1g_{\mathrm{att}},\qquad
  \|g_{\mathrm{att}}\|_2=1,\qquad
  L_{\mathrm{att}}=g_{\mathrm{att}}g_{\mathrm{att}}^\top,\qquad
  \frac{\lambda_1-\lambda_2}{\lambda_1+\varepsilon_{\mathrm{att}}}
  \geq\tau_{\mathrm{gap}}>0
  \label{eq:minimal-attachment-line-gauge}
\end{equation}
```

### Acceptance test
- [ ] abs(norm(g_att)-1) <= tol.
- [ ] ||L_att^2-L_att||_F <= tol 및 |tr(L_att)-1| <= tol.
- [ ] eigensolver raw vector를 임의 scale로 바꿔도 normalize 이후 package accept/reject가 동일.
- [ ] g_att -> -g_att에서 L_att, opposite pair, feature와 decoder 결과가 동일.

### Minimal V0 scope 영향
정규화 한 줄과 fixture만 추가한다. 학습 출력, runtime cost, solver 및 module 수는 변하지 않는다.

## RV-03 - SI--nondimensional typed map과 release proxy의 차원 일관성

- **PRIORITY:** `KEEP / NO PATCH`
- **판정:** **수용**
- **적용 시점:** V0-00 및 V0-03 유지
- **Equation label:** `eq:minimal-si-nd-typed-map`, `eq:minimal-rayleigh-unit-map`, `eq:minimal-decay-release`
- **Source line:** 195--214, 232--236, 963--999

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
각 변환은 무차원이다. 예를 들어
\[
  [\rho_A L_0^2/M_0]=1,
  \quad
  [\kappa_{\mathrm{str}}T_0^2/M_0]=1,
  \quad
  [\kappa_{\mathrm{bend}}T_0^2/(M_0L_0^2)]=1,
\]
이고 Rayleigh 계수는 \([\alpha]=T^{-1}\), \([\beta]=T\)이므로
\(\widehat\alpha=\alpha T_0\), \(\widehat\beta=\beta/T_0\)가 맞다.
Mass norm은
\[
  \|u\|_M=\sqrt{u^\top Mu},\qquad [\|u\|_M]=\sqrt{M}\,L.
\]
따라서 현재 분모
\(\sqrt{M_{r,\mathrm{ref}}}L_{\mathrm{ref}}\)와
\(\sqrt{M_{r,\mathrm{ref}}}V_{\mathrm{ref}}\)는 각각 numerator와 같은 차원을 가진다.
이전처럼 \(M_{r,\mathrm{ref}}\)를 직접 쓰면 \(M^{-1/2}\)가 남지만 현재 식은 그 오류를 제거했다.

### 판정 및 Codex action
변경하지 않는다. typed conversion과 SI/nd parity fixture만 구현한다.

### Acceptance test
- [ ] 동일 physical preset을 SI internal과 nondimensional internal로 실행했을 때 anchor trajectory/force가 tolerance 내 일치.
- [ ] 모든 converted clamp/epsilon이 package hash에 포함.
- [ ] release threshold 판정이 uniform mass scaling에 대해 불변.

### Minimal V0 scope 영향
추가 runtime 연산이 없다. conversion utility와 unit test만 필요하다.

## RV-04 - Fixed-Hessian K와 minimized bending energy의 exact tangent 불일치

- **PRIORITY:** `PATCH WORDING ONLY`
- **판정:** **감수**
- **적용 시점:** V0-01 문구 정정
- **Equation label:** `eq:minimal-structural-backend`, `eq:minimal-global-basis`
- **Source line:** 628--638, 668--680

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
\end{equation}

Free-coordinate rest operator의 generalized eigenproblem에서 eigenvalue가 낮은
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
```

### 반례 또는 차원 분석
Frozen normal \(n\)과 plane offset \(b\)에 대해 projection까지 최소화한 constraint energy는
\[
  E_{\min}(x)
  =\frac{w}{2}\left[n^\top(Ax-b)\right]^2,
  \qquad
  K_{\mathrm{tan}}=wA^\top nn^\top A.
\]
반면 package가 저장하는 fixed Hessian은
\[
  K_{\mathrm{store}}=wA^\top A.
\]
Tangential perturbation \(A\delta x=t\), \(n^\top t=0\)을 택하면
\[
  \delta^2E_{\min}=0,
  \qquad
  \delta x^\top K_{\mathrm{store}}\delta x=w\|t\|_2^2>0.
\]
따라서 generalized eigenmode는 exact physical rest-tangent mode가 아니라
predictor-frozen one-local-step PD package의 fixed-Hessian mode다.
다만 runtime global step 자체는 바로 이 fixed Hessian을 사용하므로 알고리즘 내부 모순은 아니다.

### 판정 및 Codex action
Solver와 basis를 바꾸지 않는다. source line 668의 용어만 정정하고 exact tangent claim을 금지한다.

### 권장 replacement LaTeX
```latex
V0 canonical Global basis는
\(\texttt{basis\_type=generalized\_eigen\_v1}\) 하나로 고정한다.
Free-coordinate fixed-Hessian package operator의 generalized eigenproblem에서 eigenvalue가 낮은
accepted \(r_g\)개 mode를 순서대로 사용한다.

여기서 \(K_f\)는 minimized nonlinear constraint energy의 exact rest tangent가 아니라,
predictor-frozen one-local-step PD package가 저장한 fixed Hessian이다.
```

### Acceptance test
- [ ] 수치 solver 변경 없음.
- [ ] 논문과 코드 주석에서 physical/exact rest-tangent mode라는 표현이 존재하지 않음.
- [ ] Gate A response consistency 및 Gate B strong Global-rank sweep으로 approximation을 결과 기반 검증.

### Minimal V0 scope 영향
문구와 claim boundary만 수정한다. tangent-consistent basis 재구성, 새 shell backend 또는 nonlinear iteration은 보류한다.

## RV-05 - A_ref area estimator가 불필요한 calibration scope를 만드는 문제

- **PRIORITY:** `PATCH BEFORE V0-R1`
- **판정:** **수용**
- **적용 시점:** V0-R1 이전
- **Equation label:** `eq:minimal-r1-area-decode`, `eq:minimal-area-mass`
- **Source line:** 1374--1384, 576--590

### 원문 LaTeX 전사
```latex
\begin{equation}
  \widetilde a_i=\operatorname{softplus}(\zeta_i^a)+\varepsilon_{a,\mathrm{raw}},\qquad
  a_i^0=A_{\mathrm{ref}}\frac{\widetilde a_i}{\sum_{k\in V}\widetilde a_k}
  \label{eq:minimal-r1-area-decode}
\end{equation}
로 positive total ownership을 construction으로 만족한다.
여기서 unit-typed \(A_{\mathrm{ref}}>0\)는 topology network보다 먼저
surface-valid static GS에서 versioned deterministic area estimator로 계산한다.
그 estimator는 training source mesh에서만 calibration할 수 있고 target에서는 mesh를 소비하지 않는다.
절대 면적이 없는 nondimensional branch는 식~\eqref{eq:minimal-area-mass}와 같이
\(A_{\mathrm{ref}}=1\)인 object-normalized convention을 사용한다.


\subsection{Minimal validity check}

\begin{equation}
  a_i^0>0,\qquad
  \sum_i a_i^0 \approx A_{\mathrm{asset}},\qquad
  m_i=\rho_A a_i^0,\qquad
  \sum_i m_i \approx M_{\mathrm{asset}}.
  \label{eq:minimal-area-mass}
\end{equation}

여기서 절대 \(A_{\mathrm{asset}},M_{\mathrm{asset}}\)가 알려지지 않으면
object-normalized total을 사용한다. 목적은 engineering mass identification이 아니라
Gaussian/anchor density가 달라져도 motion scale이 크게 바뀌지 않게 하는 것이다.
Nondimensional branch의 \(a_i^0,\rho_A,m_i,A_{\mathrm{asset}},M_{\mathrm{asset}}\)는
```

### 반례 또는 차원 분석
모든 area를 공통 상수 \(c>0\)로 바꾸면
\[
  a_i' = ca_i,\quad M'=cM,\quad K'=cK,\quad C'=cC,
  \quad \bar h'=c\bar h,\quad f_{\mathrm{aero}}'=cf_{\mathrm{aero}}.
\]
Core equation은
\[
  cM\ddot u+cC\dot u+cKu=c\bar h+cf_{\mathrm{aero}},
\]
이므로 clamp가 비활성인 범위에서는 \(c\)가 상쇄된다.
따라서 Minimal V0에서 중요한 것은 absolute physical area recovery보다
anchor 간 relative ownership이다. 현재 문구의 calibrated deterministic area estimator는
별도 calibration dataset, estimator ablation과 failure mode를 요구해 MC1 scope를 불필요하게 넓힌다.

### 판정 및 Codex action
A_ref를 physical area estimator가 아닌 deterministic ownership normalization constant로 정의한다. Canonical V0에서는 L_ref^2를 사용하고 별도 calibration module을 만들지 않는다.

### 권장 replacement LaTeX
```latex
여기서 unit-typed \(A_{\mathrm{ref}}>0\)는 physical surface-area estimate가 아니라
area-ownership normalization constant다. Minimal V0에서는
\[
  L_{\mathrm{ref}}=\operatorname{diameter}(x^0)>0,
  \qquad
  A_{\mathrm{ref}}=L_{\mathrm{ref}}^2
\]
로 package build 전에 결정한다. Object-normalized nondimensional geometry에서
\(L_{\mathrm{ref}}=1\)이면 \(A_{\mathrm{ref}}=1\)이다.
Training source mesh와 target static GS에서 별도의 absolute-area calibration estimator를 만들지 않는다.
식~\eqref{eq:minimal-area-mass}에서 absolute area가 알려지지 않은 경우
\(A_{\mathrm{asset}}:=A_{\mathrm{ref}}\),
\(M_{\mathrm{asset}}:=\rho_AA_{\mathrm{ref}}\)를 package convention으로 사용한다.
```

### Acceptance test
- [ ] Area logits가 항상 positive이고 sum_i a_i^0 = A_ref.
- [ ] A_ref를 c배 한 unclamped diagnostic run에서 trajectory가 tolerance 내 동일.
- [ ] force/area/covariance clamp가 scale-invariance fixture를 방해하면 clamp 활성률을 별도 실패로 기록.
- [ ] target setup에서 source mesh 또는 calibrated area estimator 호출이 없음.

### Minimal V0 scope 영향
별도 estimator와 calibration study를 제거하므로 구현 범위가 감소한다. MC1은 relative ownership과 response robustness에 집중한다.

## RV-06 - V0-R1 version contract는 있으나 canonical config 값은 아직 미동결

- **PRIORITY:** `DEFERRED DECISION / BLOCK TRAINING UNTIL SET`
- **판정:** **후속 보류**
- **적용 시점:** V0-R1 학습 시작 직전
- **Equation label:** `eq:minimal-r1-version-contract`, `eq:minimal-r1-anchor-candidate`
- **Source line:** 1323--1343, 1386--1390

### 원문 LaTeX 전사
```latex
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
선택한 routine과 cardinality/radius/normalization은 object-disjoint development split 전에 동결하고
asset별 retuning을 금지한다.

Training-only source mesh에는 anchor-to-surface map과 label generator
\(\mathcal Y^{\nu_{\mathrm{label}}}\)를 적용해 relation, close-layer/shortcut과 area target
\((y_{ik}^{\mathrm{rel}},y_{ik}^{\mathrm{layer}},a_i^\star)\)를 만든다. Connectivity-hop, geodesic-radius
또는 versioned side/layer rule 중 선택한 정확한 한 규칙과 모든 radius/hop/mapping tie-break를 config/hash에
저장하며, geodesic radius 하나를 모든 asset의 유일한 ground truth로 강제하지 않는다. Source mesh는
```

### 반례 또는 차원 분석
비균일 GS sampling에서 fixed \(k\)-NN은 sparse 영역에 긴 edge를 만들 수 있고,
fixed radius graph는 같은 영역의 degree를 0으로 만들 수 있다.
따라서
\[
  E_{\mathrm{cand}}^{\mathrm{kNN}}
  \neq E_{\mathrm{cand}}^{\mathrm{radius}}
\]
이며 동일 network/threshold라도 package acceptance와 response가 달라진다.
Source-mesh label도 geodesic radius와 connectivity hop이 irregular triangulation에서 다른 ground truth를 만든다.
현재 문서는 선택지를 versioning하는 framework contract로는 충분하지만,
하나의 재현 가능한 canonical V0-R1 실행값은 아직 정해지지 않았다.

### 판정 및 Codex action
현재 review에서 sampler/graph/label 값을 임의로 선택하지 않는다. V0-R1 training entry point는 필수 config field가 unset이면 fail-fast해야 하며, development split 사용 전에 한 config를 동결한다.

### Acceptance test
- [ ] nu_anchor, nu_cand, nu_label, nu_decode가 모두 non-placeholder이고 hash에 포함되지 않으면 training 시작 불가.
- [ ] 동일 config/seed에서 candidate graph와 label hash가 byte-identical.
- [ ] Test 결과를 본 뒤 threshold/radius/hop 변경을 금지하는 manifest check.

### Minimal V0 scope 영향
새 sampler나 label generator를 여러 개 구현하라는 요구가 아니다. 이미 허용한 선택지 중 정확히 하나만 canonical로 고정한다.

## RV-07 - Layer label의 positive-class polarity 미정의

- **PRIORITY:** `PATCH BEFORE V0-R1`
- **판정:** **수용**
- **적용 시점:** V0-R1 label generator 구현 전
- **Equation label:** `eq:minimal-topology-loss`
- **Source line:** 1386--1418

### 원문 LaTeX 전사
```latex
Training-only source mesh에는 anchor-to-surface map과 label generator
\(\mathcal Y^{\nu_{\mathrm{label}}}\)를 적용해 relation, close-layer/shortcut과 area target
\((y_{ik}^{\mathrm{rel}},y_{ik}^{\mathrm{layer}},a_i^\star)\)를 만든다. Connectivity-hop, geodesic-radius
또는 versioned side/layer rule 중 선택한 정확한 한 규칙과 모든 radius/hop/mapping tie-break를 config/hash에
저장하며, geodesic radius 하나를 모든 asset의 유일한 ground truth로 강제하지 않는다. Source mesh는
label 생성에만 쓰고 target setup/runtime 입력에는 들어가지 않는다. 식~\eqref{eq:minimal-attachment-line-gauge}의
in-plane gauge는 analytic이므로 orientation label/output/loss를 만들지 않고 \(\Delta\theta_i=0\)을 유지한다.

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
```

### 반례 또는 차원 분석
Decoder는 \(p_{ik}^{\mathrm{layer}}\le\tau_{\mathrm{layer}}\)인 edge만 accept한다.
따라서 intended semantics는 높은 probability가 invalid cross-layer/shortcut을 뜻해야 한다.
반대로 label을
\(y_{ik}^{\mathrm{layer}}=1\iff\text{same-layer valid}\)로 구현하면 BCE는 좋은 edge에서
\(p_{ik}^{\mathrm{layer}}\to1\)을 학습하지만 decoder는 그 edge를 제거한다.
Loss가 정상 감소해도 graph가 반대로 decode되는 치명적 implementation error가 가능하다.

### 판정 및 Codex action
positive class를 invalid cross-layer/shortcut으로 명시하고 이름/decoder를 그대로 유지한다.

### 권장 replacement LaTeX
```latex
Layer target의 positive class는 다음으로 고정한다.
\[
  y_{ik}^{\mathrm{layer}}=1
  \iff
  \{i,k\}\text{가 invalid cross-layer 또는 geometric shortcut candidate다.}
\]
따라서
\[
  p_{ik}^{\mathrm{layer}}
  =\Pr\!\left(y_{ik}^{\mathrm{layer}}=1\mid\phi_i,\phi_k,\chi_{ik}\right)
\]
이며 frozen decoder는 \(p_{ik}^{\mathrm{layer}}\leq\tau_{\mathrm{layer}}\)인 candidate만
same-layer relation acceptance로 넘긴다.
```

### Acceptance test
- [ ] Synthetic same-layer valid pair: y_layer=0.
- [ ] Synthetic close front/back shortcut pair: y_layer=1.
- [ ] p_rel high, p_layer low이면 accept; p_layer high이면 reject.
- [ ] Label enum/string이 model output metadata와 decoder metadata에서 동일.

### Minimal V0 scope 영향
Label enum과 문장만 고정한다. architecture, loss, decoder threshold 및 runtime module은 변하지 않는다.

## RV-08 - F_safe의 reflection 및 singular-value clamp 규칙 미완전 정의

- **PRIORITY:** `PATCH BEFORE V0-04`
- **판정:** **수용**
- **적용 시점:** V0-04 transport 구현 전
- **Equation label:** `eq:minimal-shell-extension`, `eq:minimal-gaussian-transport`
- **Source line:** 1161--1170, 1212--1247

### 원문 LaTeX 전사
```latex
\begin{equation}
  F_{j,\mathrm{shell}}^t
  =
  F_{j,\parallel}^t
  +n_{j,\mathrm{fit}}^t(n_{j,\mathrm{fit}}^0)^\top
  \label{eq:minimal-shell-extension}
\end{equation}

을 만든다. 이 상대 정의는 identity에서 \(F_{j,\mathrm{shell}}^0=I\),
proper rigid motion \(Q\)에서 \(F_{j,\mathrm{shell}}^t=Q\)를 정확히 재현한다.

를 모두 요구한다. 따라서 reflection, rank-1 collapsed support와 numerical rank-3 ambiguity를
rigid 성공으로 간주하지 않는다. Affine path가 valid하면 polar/SVD에서 reflection과 excessive scale을
bound한 map을 \(F_{j,\mathrm{safe}}^t\)라 하고, affine path가 실패했지만 위 rigid path가 성공하면
\(F_{j,\mathrm{safe}}^t=R_{j,\mathrm{rigid}}^t\)로 둔다.

Binding package는 \texttt{tangent\_fit\_version}, ordered-neighbor/weight version,
\(\kappa_{\mathrm{MLS}}\), \(\kappa_{J,\mathrm{MLS}}\),
normal floor, rigid-fallback ID와
\(\tau_K,\tau_{K,\mathrm{ratio}},\tau_{K,\mathrm{plane}},\tau_{K,\mathrm{res}}\),
reflection/scale bound, covariance eigenvalue bound와
development에서 동결한 renderer fallback-rate cap \(\tau_{\mathrm{rigid}}\)을 저장한다.
Run의 required Gaussian--frame transport attempt 수에 대한 successful rigid-fallback event 비율을
\(r_{\mathrm{rigid}}\)로 기록한다. Rigid fallback이 성공하고
\(r_{\mathrm{rigid}}\leq\tau_{\mathrm{rigid}}\)이면 fallback-marked renderer 결과를 사용할 수 있다.
Rigid fit 자체가 실패하거나 cap을 넘으면 식~\eqref{eq:minimal-attempt-success-contract}의
in-envelope method failure다. 이 fallback은 식~\eqref{eq:minimal-surface-tangent-fit}의
surface/aero consumer에는 적용하지 않는다. Gaussian mean/covariance는

\begin{equation}
\begin{aligned}
  \mu_j^t
  &=
  \bar x_j^t+F_{j,\mathrm{safe}}^t(\mu_j^0-\bar x_j^0),\\
  \widetilde\Sigma_j^t
  &=
  F_{j,\mathrm{safe}}^t\Sigma_j^0(F_{j,\mathrm{safe}}^t)^\top
  =Q_j^t\operatorname{diag}(\lambda_{j1}^t,\lambda_{j2}^t,\lambda_{j3}^t)(Q_j^t)^\top,\\
  \bar\lambda_{jk}^t
  &=\operatorname{clip}(\lambda_{jk}^t;\lambda_{\min},\lambda_{\max}),
  \qquad k\in\{1,2,3\},\\
  \Sigma_j^t
  &=
  Q_j^t\operatorname{diag}(\bar\lambda_{j1}^t,\bar\lambda_{j2}^t,\bar\lambda_{j3}^t)(Q_j^t)^\top .
\end{aligned}
  \label{eq:minimal-gaussian-transport}
\end{equation}
```

### 반례 또는 차원 분석
\(F_{j,\mathrm{shell}}^t\)의 singular values가
\((100,1,0.01)\)이면 covariance는 해당 축에서 대략
\((10^4,1,10^{-4})\)배 변할 수 있어 splat explosion 또는 needle artifact를 만든다.
현재 ``polar/SVD에서 reflection과 excessive scale을 bound''라는 문장만으로는
singular-value clip, log-scale clip, condition-only clip, reflection 축 반전 등
서로 다른 구현이 모두 허용된다. Mean과 covariance 결과가 구현체마다 달라질 수 있다.

### 판정 및 Codex action
기존 3x3 SVD와 rigid fallback만 사용해 F_safe를 식으로 닫는다. Reflection은 affine failure로 처리하고 기존 proper-Kabsch fallback으로 넘긴다.

### 권장 replacement LaTeX
```latex
Valid affine candidate에 대해
\begin{equation}
\begin{aligned}
  F_{j,\mathrm{shell}}^t
  &=U_{F,j}^t
    \operatorname{diag}(\sigma_{F,j1}^t,\sigma_{F,j2}^t,\sigma_{F,j3}^t)
    (V_{F,j}^t)^\top,\\
  \bar\sigma_{F,jk}^t
  &=\operatorname{clip}(\sigma_{F,jk}^t;\sigma_{\min},\sigma_{\max}),
  \qquad 0<\sigma_{\min}\leq1\leq\sigma_{\max},\\
  F_{j,\mathrm{safe}}^t
  &=U_{F,j}^t
    \operatorname{diag}(\bar\sigma_{F,j1}^t,\bar\sigma_{F,j2}^t,\bar\sigma_{F,j3}^t)
    (V_{F,j}^t)^\top .
\end{aligned}
  \label{eq:minimal-affine-safe-map}
\end{equation}
Singular values는 내림차순으로 정렬한다.
\(\det(U_{F,j}^t(V_{F,j}^t)^\top)\leq0\)이면 affine path를 invalid로 판정하고
식~\eqref{eq:minimal-renderer-rigid-fallback}의 기존 proper-rigid fallback을 시도한다.
Reflection을 singular-vector sign 변경으로 조용히 보정하지 않는다.
```

### Acceptance test
- [ ] Identity: F_safe=I, affine clamp와 covariance clamp 모두 비활성.
- [ ] Proper rigid Q: F_safe=Q.
- [ ] Bound 안의 in-plane affine: F_safe=F_shell.
- [ ] det(F_shell)<0: affine invalid 후 rigid fallback 또는 method failure.
- [ ] Extreme scale: singular values가 [sigma_min,sigma_max] 안에 있고 covariance SPD 유지.
- [ ] Affine clamp event rate를 trace에 기록.

### Minimal V0 scope 영향
새 stage가 아니다. 이미 요구된 SVD와 rigid fallback 안에서 singular-value clip을 명시한다. Surface/aero physics에는 적용하지 않는다.

## RV-09 - Log-PSD metric에 dimensional PSD를 직접 입력

- **PRIORITY:** `PATCH BEFORE EVALUATION FREEZE`
- **판정:** **수용**
- **적용 시점:** Gate A/B 평가 스크립트 동결 전
- **Equation label:** `eq:minimal-log-psd-errors`
- **Source line:** 1567--1575

### 원문 LaTeX 전사
```latex
를 계산한다. 동일 sampling/window와 development에서 동결한 frequency bin \(B\)에서
\(\delta_s(f)=\log(P_s(f)+\varepsilon_P)-\log(P_{\mathrm{ref},s}(f)+\varepsilon_P)\)라 하고
\begin{equation}
  e_{\mathrm{PSD,RMS},s}
  =\left(|B|^{-1}\sum_{f\in B}\delta_s(f)^2\right)^{1/2},\qquad
  e_{\mathrm{PSD,L1},s}
  =|B|^{-1}\sum_{f\in B}|\delta_s(f)|
  \label{eq:minimal-log-psd-errors}
\end{equation}
```

### 반례 또는 차원 분석
Velocity PSD의 차원은 one-sided PSD convention에서
\[
  [P]=\frac{(L/T)^2}{1/T}=L^2/T.
\]
따라서 \(\log P\)는 dimensional quantity의 로그다.
Velocity 단위를 m/s에서 cm/s로 바꾸면 \(P'=10^4P\)가 된다.
\(\varepsilon_P\)를 같은 비율로 변환하지 않으면 low-power bin의
log difference가 단위 선택에 따라 달라진다.

### 판정 및 Codex action
PSD를 frozen unit-typed scale로 나눈 뒤 dimensionless epsilon과 log를 적용한다.

### 권장 replacement LaTeX
```latex
동일 sampling/window와 development에서 동결한 frequency bin \(B\)에서
unit-typed \(T_{\mathrm{PSD,ref}}>0\)와 dimensionless
\(\widehat\varepsilon_P>0\)를 evaluation config에 저장하고
\begin{equation}
\begin{aligned}
  P_{\mathrm{scale}}
  &=V_{\mathrm{ref}}^2T_{\mathrm{PSD,ref}},\\
  \widehat P_s(f)&=P_s(f)/P_{\mathrm{scale}},\qquad
  \widehat P_{\mathrm{ref},s}(f)=P_{\mathrm{ref},s}(f)/P_{\mathrm{scale}},\\
  \delta_s(f)
  &=\log(\widehat P_s(f)+\widehat\varepsilon_P)
    -\log(\widehat P_{\mathrm{ref},s}(f)+\widehat\varepsilon_P),\\
  e_{\mathrm{PSD,RMS},s}
  &=\left(|B|^{-1}\sum_{f\in B}\delta_s(f)^2\right)^{1/2},\qquad
  e_{\mathrm{PSD,L1},s}
  =|B|^{-1}\sum_{f\in B}|\delta_s(f)|.
\end{aligned}
  \label{eq:minimal-log-psd-errors}
\end{equation}
로 계산한다.
```

### Acceptance test
- [ ] 동일 signal을 m/s와 cm/s로 표현해도 PSD error가 tolerance 내 동일.
- [ ] Zero/near-zero power bin에서 finite result.
- [ ] T_PSD_ref, epsilon, window와 frequency bins가 evaluation hash에 포함.

### Minimal V0 scope 영향
Runtime physics에는 영향이 없다. 평가 후처리와 manifest field만 수정한다.

## RV-10 - Surface/transport tangent의 rank=2가 numerical rank로 정의되지 않음

- **PRIORITY:** `PATCH NOW`
- **판정:** **수용**
- **적용 시점:** V0-01 tangent fit 구현 전
- **Equation label:** `eq:minimal-surface-tangent-fit`, `eq:minimal-affine-weights`, `eq:minimal-affine-fit`
- **Source line:** 813--821, 1111--1123, 1146--1148

### 원문 LaTeX 전사
```latex
Setup에서는
\(\operatorname{rank}(G_{i,\mathrm{surf}}^0)=2\),
\(\kappa(G_{i,\mathrm{surf}}^0)\leq\kappa_{G,\mathrm{surf}}\),
\(\operatorname{rank}(J_i^0)=2\),
\(\kappa((J_i^0)^\top J_i^0)\leq\kappa_{J,\mathrm{surf}}\), orientation agreement와
\(\|\widetilde n_i^0\|>\tau_{n,\mathrm{surf}}>0\)를 hard-check한다.
Runtime에는 \(\operatorname{rank}(J_i^t)=2\),
\(\kappa((J_i^t)^\top J_i^t)\leq\kappa_{J,\mathrm{surf}}\)와
\(\|\widetilde n_i^t\|>\tau_{n,\mathrm{surf}}\)를 요구한다.

\begin{equation}
\begin{aligned}
  &\omega_{ji}>0,\qquad
  \sum_{i\in\mathcal N_j}\omega_{ji}=1,\\
  &\bar x_j^s=\sum_{i\in\mathcal N_j}\omega_{ji}x_i^s
  \quad(s\in\{0,t\}),\qquad
  \xi_{ji}=(T_j^0)^\top(x_i^0-\bar x_j^0),\\
  &\sum_i\omega_{ji}\xi_{ji}=0,\qquad
  G_j^0=\sum_i\omega_{ji}\xi_{ji}\xi_{ji}^\top,\qquad
  \operatorname{rank}(G_j^0)=2,\quad \kappa(G_j^0)\leq\kappa_{\mathrm{MLS}}.
\end{aligned}
  \label{eq:minimal-affine-weights}
\end{equation}

\(J_j^0\)와 \(J_j^t\)는 column rank 2이고
\(\kappa((J_j^s)^\top J_j^s)\leq\kappa_{J,\mathrm{MLS}}\),
\(s\in\{0,t\}\)여야 한다.
```

### 반례 또는 차원 분석
\[
  J=\begin{bmatrix}1&0\\0&10^{-14}\\0&0\end{bmatrix}
\]
는 exact arithmetic에서는 rank 2지만
\[
  J^\top J=\operatorname{diag}(1,10^{-28}),
  \qquad \kappa(J^\top J)=10^{28}
\]
이므로 numerical rank-1이다. Library별 \texttt{rank()} tolerance가 다르면
inverse 호출, package rejection과 frame failure가 플랫폼마다 달라질 수 있다.
Condition cap이 있더라도 inverse 전에 동일한 threshold semantics를 적용해야 한다.

### 판정 및 Codex action
tangent_fit_v1에 shared relative+absolute numerical-rank contract를 추가하고 G/J inverse 전에 적용한다.

### 권장 replacement LaTeX
```latex
Tangent-fit numerical rank는 다음 shared convention으로 고정한다.
\begin{equation}
  \operatorname{rank}_{\tau,\sigma_{\mathrm{abs}}}(A)
  :=\#\left\{k:
  \sigma_1(A)\geq\sigma_{\mathrm{abs}},\quad
  \sigma_k(A)/\sigma_1(A)\geq\tau\right\}.
  \label{eq:minimal-tangent-numerical-rank}
\end{equation}
Setup/runtime에서 \(J\in\mathbb R^{3\times2}\)는
\[
  \operatorname{rank}_{\tau_J,\sigma_{J,\mathrm{abs}}}(J)=2
\]
를 요구한다. SPD \(G\in\mathbb R^{2\times2}\)는 eigenvalue를
\(\lambda_1\geq\lambda_2\geq0\)로 정렬하고
\[
  \lambda_1\geq\lambda_{G,\mathrm{abs}},\qquad
  \lambda_2/\lambda_1\geq\tau_G
\]
를 통과한 뒤에만 \(G^{-1}\)을 계산한다.
모든 threshold는 \texttt{tangent\_fit\_v1} package hash에 포함한다.
```

### Acceptance test
- [ ] Near-rank-1 synthetic J가 모든 플랫폼에서 reject.
- [ ] Well-conditioned planar J가 accept.
- [ ] G inverse는 numerical-rank/condition check 이후에만 호출.
- [ ] CPU/GPU 또는 두 linear algebra backend에서 동일 accept/reject.

### Minimal V0 scope 영향
기존 SVD/eigendecomposition threshold를 명시할 뿐 fallback이나 새 solver를 추가하지 않는다. Failure semantics는 기존 fail-fast를 유지한다.

## RV-11 - Canonical implementation specification과 논문 본문의 exposition 분리

- **PRIORITY:** `PAPER-STAGE ONLY`
- **판정:** **감수**
- **적용 시점:** V0-05 이후 논문 원고 작성
- **Equation label:** 해당 없음; source line 기준
- **Source line:** 76--87, 1318--1323

### 원문 LaTeX 전사
```latex
\begin{tcolorbox}[colback=CoreLight,colframe=CoreBlue,title=Canonical method/contract freeze,fonttitle=\bfseries]
이 문서는 첫 CG 논문의 method equation과 implementation contract를 소유하고 동결하는
Minimal V0 canonical source다. 논문의 최종 claim과 연구 경로는 이 동결과 구분하며,
Gate A/B/C evidence가 모이는 V0-05에서만 별도로 동결한다.
구현 checklist는 이 문서의 stable label을 참조하는 파생 실행 문서이며,
독립적인 두 번째 수식 source가 아니다.
목표는 engineering-grade thin-shell simulation이 아니라
\emph{visually plausible, temporally stable, controllable wind animation}과
실측 quality--latency 이득이다.
이전 full design의 objective normal-curvature closure, patch-force KKT/Schur,
exact reaction ledger, same-frame corrector와 confidence calibration은
design archive로 동결하며 V0 구현 요구사항이 아니다.

\section{Topology distillation contract}

\subsection{V0-R1 versioned executable package}

Topology distillation은 runtime network가 아니라 setup용 V0-R1 module 하나다. 그 실행 계약을
\begin{equation}
```

### 반례 또는 차원 분석
현재 문서는 구현 계약으로는 강하지만, MC1의 executable topology model이 source line 1318 이후에 등장한다.
같은 순서를 논문 본문에 사용하면 reviewer는 핵심 method보다 unit typing, failure denominator,
hash와 fallback contract를 먼저 읽게 되어 시스템 명세서로 인식할 가능성이 높다.
이는 수식 오류가 아니며 canonical source의 역할과는 충돌하지 않는다.

### 판정 및 Codex action
Canonical source는 수정하지 않는다. V0-05 이후 별도 paper draft에서만 exposition을 재배치한다.

### 권장 replacement LaTeX
```latex
권장 paper-body 순서:
1. Problem statement와 MC1/MC2
2. Static GS에서 sparse scaffold distillation
3. Fixed-Hessian structural package와 Global/Local bases
4. Load-conditioned selection과 coupled solve
5. Anchor-to-Gaussian transport
6. Gate A/B/C 및 wall-clock Pareto
7. Hash, failure code, typed mapping과 상세 fixture는 supplementary
```

### Acceptance test
- [ ] Canonical source hash와 method labels는 유지.
- [ ] Paper draft가 canonical source를 두 번째 독립 수식 source로 재정의하지 않음.

### Minimal V0 scope 영향
구현, solver, module 및 실험 스코프 영향이 없다. 논문 서술 단계에서만 처리한다.

## 3. 최종 패치 체크리스트
- [ ] RV-02: g_att unit normalization 및 line-projector fixture.
- [ ] RV-04: fixed-Hessian package operator로 용어 정정; solver/basis 변경 금지.
- [ ] RV-05: A_ref calibrated estimator 제거, L_ref^2 ownership convention.
- [ ] RV-06: canonical R1 config unset 시 training fail-fast.
- [ ] RV-07: y_layer=1을 invalid cross-layer/shortcut으로 고정.
- [ ] RV-08: F_safe SVD/reflection/scale rule 완전 정의.
- [ ] RV-09: PSD nondimensionalization 후 log metric.
- [ ] RV-10: shared numerical-rank semantics를 inverse 전에 적용.
- [ ] RV-01/RV-03: 현재 식 유지.
- [ ] RV-11: canonical source가 아니라 별도 paper draft에서만 처리.

## 4. Definition of review completion
- [ ] Source compiles with XeLaTeX without undefined control sequences or duplicate labels.
- [ ] Patch diff contains only listed equation/text/config/test changes.
- [ ] No new solver, runtime module, backend branch or asset class is introduced.
- [ ] V0-01/V0-04/V0-R1/evaluation milestone ownership is preserved.
- [ ] Package/run hash fields are updated only where a new threshold or contract field is explicitly added.
- [ ] Gate A/B/C and deferred-module policy remain unchanged.

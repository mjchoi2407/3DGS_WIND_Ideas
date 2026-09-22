# Codex 실행 지시서 v3 — 기존 검산 유지, FP32 중심 혼합 정밀도와 FP64 대조 실험

작성일: 2026-09-16
입력 근거: `dual_gpu_rtx5070_v2.zip`, `dual_gpu_gtx1080ti_v2.zip`, `dual_gpu_v2_review_2026-09-16.md`
성격: 현재 저장소에서 최소 구현과 실제 실행을 수행하기 위한 작업 계약. 이 문서 자체는 새 GPU 실행 결과가 아니다.

## 0. 목표와 이전 지시서와의 관계

이번 목표는 **기존 물리 문제·최종 허용오차·독립 FP64 검산을 유지하면서, 비싼 계산의 가능한 많은 부분을 FP32로 수행하여 합격 데이터 생성 총시간을 줄이는 것**이다.

FP32 변수 개수나 명목상 FP32 비율을 최대로 만드는 것이 목표가 아니다. FP32를 더 많이 써도 반복·검증·fallback 비용 때문에 느려지면 채택하지 않는다. FP64 기준을 낮춰 성공으로 만드는 것도 금지한다.

최적화 목적은 다음과 같다.

\[
\min_{\pi} T_{\mathrm{valid}}(\pi),\qquad
\text{subject to original acceptance/audit predicates and finite outputs}.
\]

여기서 \(\pi\)는 정밀도 경계·허용된 선형 풀이 전략·명시적 실행 설정이다. \(T_{\mathrm{valid}}\)에는 후보 시도, 원래 방정식 잔차 확인, 보정, fallback, 독립 검산, 생산 저장을 포함한다. 초기화/컴파일과 상세 진단 기록은 별도로도 보고한다.

이 문서는 v2 후속 작업 범위를 대체한다. 이전의 “새 혼합 경로를 만들지 말라”는 제한은 **M1/M2를 위한 최소 선형 전략 어댑터**에 한해 해제한다. 새 물리 모델, 새 적분기, 새로운 전처리기 계열, 범용 분산 autotuner를 만드는 허가는 아니다. 기본 생산 경로와 `training_eligible`은 자동 변경하지 않는다.

계획·요약만 작성하고 끝내지 말고, 가능한 범위의 최소 구현·실측·실패 기록·재현 명령까지 완성한다. 접근 불가능한 PC/미구현 변형의 수치를 추정하지 않는다.

## 1. v2에서 확정된 출발점과 미확정 사항

- RTX 5070/W1: FP64 `256/256/256`의 solver+audit 중앙값 20.414296 s, `32/32/256`은 19.007939 s. 3프레임 192스텝, GMRES 누계 10,630회, rebuild 누계 13회, retry 0회.
- GTX 1080 Ti: `64/64/64` 후보에 채택 근거 없음. 이번 기준은 `256/256/256`이다.
- RTX 5070/W1/32·32·256 Graph fixed-work: FP64 hi/lo assembled_force 1776.619 us, 보정 FP32 hi/lo 181.248 us. 약 9.80배는 **힘 평가 한 연산군의 측정값**이지 전체 solver 가속이 아니다.
- 보정 경로는 `stable_metric_pair`. W1 동일 상태의 힘 차이는 L2 약 9.74752e-7 N, 최대 성분 약 1.74794e-7 N이다. 이것은 Newton 종료 잔차와 다르다.
- W1에서 어떤 선형 연산이 시간을 지배하는지, 보정 FP32 HVP의 정확도와 시간, 혼합 선형 풀이의 실제 수렴 시간은 아직 확인되지 않았다.
- C0는 부과 바람 0인 대조군. W1은 원본 forcing index 60/61/62, 물리 시각 3.000~3.050 s의 3프레임이다. 소스에서 현재 값과 provenance를 다시 확인한다.
- 기존 회귀/교차 장치 정확도 예산이 미정인 경우 `budget_not_defined`를 유지한다. 이 상태는 explicit 성능·기존 검산 실험을 막지 않으며 생산 승격은 보류한다.

5070은 v2의 `32/32/256`을 **실험 reference**로, 1080 Ti는 `256/256/256`을 reference로 한다. v2의 원래 설정도 기록으로 보존한다. GPU별 최적 정밀도 경계도 달라도 된다.

## 2. 변경 불가 조건과 제한적으로 허용하는 변경

### 2.1 고정할 것

P3 요소, mesh·구적·재료·두께·penalty, 외력·구속, Newmark 공식, 물리 시각, dt·재시도 정책, 공식 힘/변위 허용오차, Newmark 내부 안전 여유, 기존 EW 선형 목표 정책, line-search acceptance, 최대 Newton 정책, 독립 검산 및 에너지/갱신식 장부 기준을 고정한다.

허용오차는 채팅의 반올림 숫자를 하드코딩하지 말고 실제 policy·runtime·검산 소스에서 추출해 `acceptance_contract.json`에 저장한다. 각 기준마다 source hash, 함수, 단위, 자유 DOF, norm, 상대항 분모, 비교 연산, 내부/공식 구분을 명시한다.

새로운 epsilon/allclose, baseline 편차의 임의 배수, dtype epsilon으로 최종 합격 기준을 대체하지 않는다. FP32에서 작은 중력이 묻혀 움직이지 않는 것을 성공으로 처리하지 않는다.

### 2.2 허용되는 최소 변경

- FP64 상태·원래 잔차를 소유하는 Newmark에 M1/M2 선형 전략을 별도 옵션으로 연결.
- FP32 내부 보정 목표와 제한된 반복 예산. 이것은 원래 Newton 선형 목표/최종 기준을 완화하는 것이 아니다.
- 정확히 역변환되는 행·열 스케일링 및 RHS 정규화.
- FP64 참 잔차에 의한 보정·fallback.
- 별도 FP64 대조 실험에서 기존 전처리 행렬의 재구축 빈도, HVP launch 설정, 선택적 TwoProduct-FMA 변경. 승인 범위는 §10을 따른다.

M1/M2 도입으로 선형 내부 알고리즘은 달라질 수 있지만 최종 linear-success 의미는 달라지지 않는다. 기본 경로·독립 reference·실험 모듈을 별도로 유지한다.

기존 일괄 dtype source 변환을 reference 패키지에 적용하지 않는다. low 패키지는 별도 namespace/hash로 로드하고 master 상태·A64·auditor가 low 모듈을 참조하지 않는지 확인한다. 어댑터가 상위 `after_linear`에 반환하는 성공 플래그와 잔차 통계는 최종 A64 참 잔차에서 채우며, low 내부의 예상 잔차를 대입하지 않는다. 버린 inner 반복·보정·fallback 호출도 누계에 포함한다.

## 3. 실제 구현 의미: 변위가 아니라 가속도 보정이다

v2 동결 소스에서 다음을 확인했다. 현재 checkout과 다르면 먼저 차이를 설명하고 source hash를 고정한다.

- `teacher/p3_shell_resident_stepper.py`: `ResidentShellStepper.action`, `_evaluate`, `_newton`.
- `teacher/resident_step_kernels.py`: `tangent_result`, `residual`, `ew_tolerance`, `trial`.
- `teacher/p3_shell_resident.py`: `evaluate(u_hi,u_lo)`와 `hvp(u,direction)`의 입력 의미가 다르다.

현재 참 연산자 구현은 자유 DOF에서

\[
A_{64}d=M_{64}d+c K_{64}(u)d,\qquad c=\tfrac14\Delta t^2,
\]
\[
b_{64}=f_{\mathrm{int},64}(u_{hi},u_{lo})+f_{\mathrm{held},64}-M_{64}a,
\qquad A_{64}d=b_{64}.
\]

여기서 \(d\)는 **가속도 보정**, \(K_{64}d\)는 현재 HVP 구현의 출력이다. line-search 배율 \(\lambda\)에서

\[
a_{trial}=a+\lambda d,\qquad
u_{trial}=u+\lambda c d
\]

를 기존 hi/lo 연산으로 수행한다. 변위 해라고 오인하여 `u += d`로 적용하면 안 된다. 동일 이전 상태를 기준으로 한 위치/속도 보정의 관계는

\[
\delta u=c\,\delta a,\qquad \delta v=\tfrac12\Delta t\,\delta a
\]

이다. 이 관계를 상태 오차 진단에 이용할 수 있지만 별도 합격 기준으로 쓰지 않는다.

현재 baseline HVP는 `uh`를 받으며 힘 평가는 `uh,lo`를 받는다. 이 차이를 숨기거나 baseline에 임의로 lo 항을 추가하지 않는다. **이 실험의 A64는 실제 기준 action**이다. 그것이 완전한 수학적 Jacobian과 일치한다고 측정 없이 선언하지 않는다.

## 4. P0 — W1 프로파일과 대표 선형계 저장

### 4.1 실제 비용 분해

5070/W1 reference에서 일반 실행 최소 3회와 별도 진단 실행을 수행한다. GPU·CPU 중첩 시간을 더하지 않는다. solver/audit를 구분하며 다음 항목을 가능한 기존 NVTX/Graph 범위로 나눈다.

- 비선형 힘·기하·조립, line-search 재평가.
- 기준 HVP 및 mass action.
- 전처리 행렬 조립, 수치 분해, cuDSS apply.
- Arnoldi 직교화·내적·norm, 작은 Hessenberg/Givens/backsolve, 벡터 갱신.
- 참 선형 잔차 확인, 상태 갱신, 독립 검산, 전송/저장.

C0의 옛 비중을 W1에 재사용하지 않는다. 프로파일러 실행 시간을 생산 성능으로 보고하지 않는다. 지원되지 않는 프로파일 기능 때문에 일반 측정을 중단하지 않는다. 모든 커널 뒤에 host synchronization을 넣지 않는다.

각 Newton 선형 solve의 frame/substep/Newton index, baseline eta, norm(b), original true-residual target, GMRES total/restart 수, P 종류(rest/current), factor generation·age, rebuild 여부를 device 버퍼에 기록하고 저장 경계에서 읽는다. 기존 마지막 Newton 카운터를 전체 누계로 오독하지 않는다.

### 4.2 실제 선형계 3개 선정

단계별 모든 상태를 대량 저장하지 않는다. 진단 로그에서 다음 3개를 결정해 타깃 replay로 확보한다.

- L0: 정상적인/중앙 정도 비용의 선형계.
- L1: W1에서 GMRES 비용이 큰 선형계.
- L2: 작은 RHS 또는 엄격한 eta로 정확도 회복이 중요한 선형계. L1과 겹치면 다른 eligible 선형계를 선택.

선정 규칙·인덱스를 먼저 기록하고 FP32가 성공한 사례만 골라 보고하지 않는다. 먼저 L1에서 구현 smoke를 하고, 유망 후보를 L0/L1/L2 전체에 검증한다.

필수 저장: `uh,lo,a`, master hi/lo 상태, held force와 물리 시각, 자유 DOF ids, M 및 실제 basis/구적 arrays, dt와 c, b64, 초기 x, eta/원래 target, rest/current P64의 CSR 원값과 generation·age, 실제 사용한 입력 방향 벡터와 가능하면 baseline Krylov 방향 일부, 기준 해·참 잔차. opaque factor handle을 snapshot으로 저장하지 않는다. 저장 CSR에서 재분해하고 true action 대비 확인한다. full A64를 dense로 조립할 필요는 없다.

적어도 b 방향, 실제 Krylov/보정 방향, 고정 seed 방향 3개를 포함한다. seed 방향만으로 HVP를 평가하지 않는다. 프로파일에 없는 Newton 상태를 사후 그럴듯하게 합성하지 않는다.

## 5. P1 — 연산자 정확도와 정밀도 경계 확인

### 5.1 기존 경로 재사용

다음 구현을 먼저 확인한다.

- `resident_adaptive_precision.py`: FP32 보조 direct correction을 제한 횟수 적용하고 A64 잔차로 확인한 후 FP64 GMRES로 fallback. 이것은 아래 M2 전체 FP32 Krylov 보정과 같지 않다.
- `resident_gauss_mixed.py`: Gauss의 FP64 residual/FP32 inner 보정 기반. 스케일링·통계·코드 일부는 재사용하되 3-stage 블록식/시간 적분기 자체를 Newmark에 이식하지 않는다.
- `resident_gauss_equilibration.py`: 기존 power-of-two row/column 스케일링. source dtype·적용 순서·스케일 수명 확인.
- `teacher_precision_compare.py`, `strain_precision_specialization.py`: `stable_metric_pair`의 실제 적용 모듈.

FP32 힘이 보정되어도 HVP가 같은 보정을 자동으로 받는다고 가정하지 않는다. HVP 경로의 실제 kernels와 변환된 소스를 확인한다. mixed kernel 내 dtype, state conversion, geometry, constitutive, HVP, mass term, assembly, vector, reductions, factor/apply 각각을 `precision_map.json`에 적는다.

원본 상태/데이터는 FP64로 보존한다. low view만 별도 생성한다. hi+lo를 먼저 단일 FP32로 합쳐 작은 성분을 버리지 않는다. baseline HVP가 lo를 사용하지 않는다는 원래 의미와 low state 저장의 정밀도는 구분한다.

### 5.2 출력 오차 정의

동일한 FP64 입력 방향 d에 대하여

\[
y_{ref}=A_{64}(d),\quad d_q=\operatorname{cast}_{64}(\operatorname{cast}_{32}d),
\]
\[
y_{input}=A_{64}(d_q),\quad y_{low}=\operatorname{cast}_{64}(A_{32}(\operatorname{cast}_{32}d)).
\]

총 오차, 방향 반올림 효과, 나머지 저정밀 경로 효과를 각각

\[
e_{total}=y_{low}-y_{ref},\quad e_{input}=y_{input}-y_{ref},\quad
 e_{remaining}=y_{low}-y_{input}
\]

로 보고한다. low state/geometry 자체가 반올림되어 있다면 `e_remaining`을 순수 산술 오차라고 부르지 말고 `state_geometry_plus_arithmetic`으로 표기한다. 필요한 경우 동일하게 반올림된 state/coefficients를 FP64로 평가하는 추가 대조 하나로 분리한다.

각 오차 배열 e의 자유 성분 집합 F, 크기 n에 대해

\[
E_\infty=\max_{j\in F}|e_j|,\quad E_2=\sqrt{\sum_{j\in F}e_j^2},\quad
E_{RMS}=E_2/\sqrt n,\quad E_{rel}=E_2/\|y_{ref}\|_2.
\]

분모가 0이면 relative=null, 절대값과 `zero_reference`를 기록한다. diagnostic epsilon을 물리 기준에 더하지 않는다. NaN/Inf는 문자열이 아니라 실제 값으로 검사한다.

K·mass·combined A의 오차를 따로 본다. combined A의 mass 항이 커서 HVP 차이가 상대적으로 작게 보일 수 있다. 기존 조립 P와 A64는 근사/갱신 시점이 다를 수 있으므로 원래부터 P=A라고 가정하지 않는다.

### 5.3 평가 역할

FP32 inner operator는 정확한 A64와 작게 다를 수 있다. 이 자체를 `1e-10 relative HVP` 같은 FP64용 문턱으로 무조건 탈락시키지 않는다. **최종 판단은 동일 A64 참 잔차로의 수렴과 총시간**이다. 반대로 출력이 finite이거나 상대 HVP 오차가 작다는 이유만으로 합격 처리하지 않는다.

force32 오차도 기존 데이터와 동일 정의로 남긴다. 공식 힘 norm과 같은 norm으로 \(\|f_{32}-f_{64}\|/\tau_f\)를 기록할 수 있으나 이는 대체 가능성을 보는 진단값이지 Newton 잔차가 아니다.

## 6. P2 — FP32 사용 범위를 넓히는 두 필수 후보

| 경로 | 상태·비선형 힘/판정 | 선형 반복의 A action | P 분해/apply | Krylov/보정 | 최종 승인 |
|---|---|---|---|---|---|
| R64 | 기존 FP64/hi·lo | 원래 A64 | 기존 FP64 | FP64 | 기존 FP64 |
| M1 | R64 그대로 | 원래 A64 | FP32 | 우선 FP64 | 원래 A64 참 잔차 |
| M2 | R64 그대로 | 보정/검증된 A32 inner | FP32 우선 | 큰 벡터 FP32, 내적·작은 문제 FP64 | 외부 A64 참 잔차/FP64 보정 |

### 6.1 M1 — FP32 전처리, 원래 FP64 Krylov 방정식

FP64로 만든 원본 P64를 보존하고, 같은 sparsity/시점에서 FP32 P를 생성한다. FP32 분해·삼각 풀이를 FP64 GMRES의 전처리 action으로 감싼다. per-apply 입출력 cast·스케일링 비용도 포함한다.

이는 기존 adaptive32의 “근사 direct correction 1~2회 후 전부 FP64로 복귀”와 별도 후보다. 같은 precision 표시라도 동작이 다름을 보고한다.

한 Krylov solve 동안 P의 내용·분해·스케일링을 고정한다. 작은 pivot 관련 설정, 보정 pivot 수와 info는 설치된 cuDSS API에서 가능한 범위만 조회한다. 최신 문서의 기본값을 0.7.1에 그대로 가정하지 않는다. pivot epsilon을 FP64 값으로 무작정 낮추거나 라이브러리를 업그레이드하지 않는다.

M1의 목적은 “P만 FP32여도 원래 목표에 빠르게 도달하는가”를 확인하는 보수적인 대조다. M1의 가속이 작다고 M2를 포기하지 않는다.

### 6.2 M2 — FP32 inner Krylov + FP64 residual correction

고정 Newton 선형계에서 \(x\)는 가속도 보정의 누적 해다.

\[
x_0=0,\qquad r_j=b_{64}-A_{64}x_j.
\]

원래 선형 목표를 \(\tau_{lin}\)이라 할 때,

\[
\|r_j\|_{\mathrm{original}}\le\tau_{lin}
\]

일 때만 성공한다. 현재 baseline이 \(\eta_k\|b\|_2\)를 사용한다면 그것을 그대로 사용하고, 실제 absolute항/특수조건이 있으면 그 predicate 전체를 보존한다. frozen linear에서는 저장된 eta/target, 실제 integration에서는 원래 EW 함수가 현재 FP64 상태에서 계산한 eta를 쓴다. 궤적이 달라졌는데 baseline 모든 eta를 강제로 재생하지 않는다.

각 correction에서 inner32로

\[
A_{32} e_j\simeq \operatorname{cast}_{32}(r_j)
\]

를 풀고, scale을 쓴 경우 아래 역변환 후

\[
x_{j+1}=\operatorname{update}_{64}(x_j,e_j),\qquad
r_{j+1}=b_{64}-A_{64}x_{j+1}
\]

를 다시 계산한다. master x와 residual은 FP64, master 물리 상태는 기존 FP64 hi/lo다. 상쇄/범위 검사로 필요성이 확인된 경우에만 선형 x의 보상 누적도 추가한다.

**FP32 HVP·factor·주요 벡터 연산은 inner에만 넣는다.** 첫 구현에서 dot/norm, 작은 Hessenberg·Givens·backsolve는 FP64로 유지해 직교성 오차까지 동시에 도입하지 않는다. FP32 벡터의 dot을 FP64로 누적하려면 값의 곱도 FP64 승격 후 계산하는지 기록한다. 큰 벡터 storage를 FP32로 두고 작은 reduction을 FP64로 해도 FP32 중심 후보다.

초기 탐색 값은 `inner_rtol=1e-2`, 외부 correction 최대 6회, inner iteration budget은 우선 기존 유효 cap 이하의 120회로 둔다. 이는 **비용 제한을 위한 실험 설정**이지 학습/최종 정확도 기준이 아니다. 이 제한으로 실패하면 `screening_budget_exhausted`로 명시하고 FP32의 원천 불가능을 선언하지 않는다. 잔차가 계속 잘 줄고 비용 여지가 있을 때만 별도 후보 하나에서 budget을 늘린다.

초기 설정이 원래 참 잔차까지 성공하면 `inner_rtol=1e-1`을 추가 비교해 과잉 내부 풀이를 줄인다. 내부 rtol·restart·scaling·factor age의 전 조합 sweep은 하지 않는다. 1e-3은 원인 진단상 필요한 경우에만 추가한다.

원래 target이 1e-10이어도 FP32 내부 solver 자체에 1e-10을 강요하지 않는다. 대신 FP64 참 잔차를 반복 보정하여 **바깥의 1e-10 목표는 그대로** 만족시킨다. 수렴 여부는 측정한다.

### 6.3 FGMRES를 적용해야 하는 경우와 아닌 경우

현재 `resident_gmres.py`는 left-preconditioned GMRES다. 가변 내측 solve 또는 반복 중 dtype/factor 전환을 같은 GMRES의 고정 전처리기인 것처럼 끼워 넣지 않는다.

초기 M2는 FP64 defect-correction 루프에서 **매번 독립된 inner GMRES**를 호출하고, 각 inner solve의 P를 고정한다. 이 구성 자체만으로 새 FGMRES가 반드시 필요하지 않다. FP64 fallback으로 전환할 때도 기존 Arnoldi basis를 그대로 이어 쓰지 말고 명시적으로 재시작한다.

반대로 adaptive inner solve를 하나의 바깥 Krylov preconditioner로 넣는 구조를 선택한다면 FGMRES 등 flexible 방식이 필요하다. 그 경우 최소 구현 변경과 적용 방향을 명시한다. PETSc 공식 문서는 fixed GMRES와 nonlinear/variable preconditioning을 허용하는 FGMRES를 구분한다. PETSc 자체를 새 dependency로 도입하라는 뜻은 아니다.

### 6.4 스케일링/RHS 정규화의 정확한 계산

먼저 기존 정상 경로를 재사용한다. 개선할 경우 아래와 같이 원래 방정식으로 역변환되는 정의를 사용한다. 스케일 계수 계산은 FP64 P64에서 수행한다.

\[
p_i=\max_j |P_{ij}|,\qquad
s_{r,i}=\begin{cases}2^{\operatorname{clip}(-\operatorname{round}(\log_2p_i),-60,60)}&p_i>0\\1&p_i=0,\end{cases}
\]
\[
q_j=\max_i |s_{r,i}P_{ij}|,\qquad
s_{c,j}=\begin{cases}2^{\operatorname{clip}(-\operatorname{round}(\log_2q_j),-60,60)}&q_j>0\\1&q_j=0.\end{cases}
\]

zero row/column은 scale=1로 처리해도 별도 structural flag를 남긴다. \(S_r=\mathrm{diag}(s_r), S_c=\mathrm{diag}(s_c)\)이며

\[
\widetilde P=S_rP S_c,\qquad \widetilde A(z)=S_r A(S_cz).
\]

M1에서 P만 scale할 때는

\[
\mathcal P^{-1}r=S_c\,\mathrm{solve}(\operatorname{cast}_{32}\widetilde P,\operatorname{cast}_{32}(S_r r))
\]

를 FP64 입출력에 맞게 구현한다. A64의 기준 방정식을 바꾸지 않는다.

M2에서 전체 inner action도 scale한다면 q=Sr r에 대해

\[
m=\|q\|_\infty,\quad
\alpha=2^{\operatorname{clip}(-\operatorname{round}(\log_2m),-100,100)}\quad(m>0),
\]
\[
\widetilde A_{32} z\simeq\operatorname{cast}_{32}(\alpha S_r r),\qquad
 e_{64}=S_c\operatorname{cast}_{64}(z)/\alpha.
\]

m=0이면 원래 residual=0 여부를 먼저 확인하며 불필요한 solve를 하지 않는다. 조정 전후 residual 비교와 최종 합격은 항상 **원래 단위의 b64−A64x**로 한다. clip hit, nonzero→zero cast 수, overflow, cast/scaling 시간을 기록한다. 역변환된 벡터가 representable하지 않으면 FP64로 승격한다. scalar scaling이 조건수 자체를 개선했다고 주장하지 않는다.

raw P를 매 generation에서 보존하고 이미 scaled된 P에 재구축 때 scale을 누적하지 않는다. P·scale·factor의 generation을 같은 수명으로 관리한다.

### 6.5 보정의 실제 효율과 fallback

외부 correction의 감소율은

\[
\rho_j=\|r_{j+1}\|_2/\|r_j\|_2
\]

로 계산한다. 분모 0은 성공 분기에서 처리한다. inner가 자기 목표를 못 맞췄더라도 유한한 correction이 원래 residual을 충분히 줄였는지 별도로 기록한다. 단순 inner iteration budget exhaustion과 NaN/잘못된 factor/geometry failure를 같은 오류로 취급하지 않는다.

초기 비용 보호 규칙: 원래 기준 통과 시 종료; 구조적/비유한 오류는 즉시 FP64; \(\rho\ge0.9\)가 2회 연속이거나 correction cap 소진 시 FP64 fallback. 0.9와 6회는 실험적 성능 정책이며 물리 허용오차가 아니다. 원래 잔차가 크게 증가하면 해당 시도를 중단한다.

fallback은 같은 Newton 상태·b64·원래 A64로 reference solver를 재시작한다. 첫 구현은 x=0 재시작으로 명확히 한다. low basis/FP32 residual을 참값으로 넘기지 않는다. 후보 비용+FP64 fallback 비용을 전부 합산한다. 순수 mixed 성공, mixed 후 FP64 fallback 성공, 물리 retry 사용, 전체 실패를 구분한다.

같은 factor generation에서 계속 같은 이유로 실패하면 그 generation의 추가 무익한 시도를 건너뛰는 옵션을 둘 수 있다. 기록 없이 대부분을 FP64로 실행한 뒤 “FP32 성공”이라고 보고하지 않는다.

## 7. P3 — M2 실패 원인에 맞춘 최소 정밀도 복원

L1에서 M2가 정체하면 FP32 포기 또는 전체 FP64 복귀만을 선택하지 말고, **다음 중 원인에 맞는 하나**를 먼저 비교한다. 전체 factorial 실험은 금지한다.

1. **P 문제 분리:** HVP32를 유지하고 같은 P의 분해/apply만 FP64로. 또는 stale P와 fresh P를 같은 dtype으로 비교한다. FP32 분해가 원인인지 원래 전처리 근사가 약한지 구분한다.
2. **HVP/geometry 문제 분리:** 상태 의존 고정 기하 계수만 Newton 시점에 FP64로 계산/캐시하고, 반복되는 방향 의존 HVP 계산은 FP32로 유지한다. 출력 assembly 또는 mass+cK 합산만 FP64로 복원하는 후보도 비용에 따라 대안이다.
3. **직교성 문제 분리:** 큰 Krylov 벡터 FP32를 유지하면서 reduction·재직교화의 FP64 경계를 확인한다. FP64 누적이 실제로 이미 적용돼 있다면 중복 구현하지 않는다.

기하 캐시는 같은 Newton 상태·외력/물성·mesh에서만 재사용한다. 방향 d에 따라 달라지는 도함수, 법선의 방향 미분, 재료 미분항을 “캐시” 명목으로 상수화하면 다른 HVP가 된다. 그러한 변경은 금지한다. A32 action의 alpha/beta, alias, constrained DOF, status 처리를 보존한다.

이 단계에서도 원래 A64를 oracle로 동결한다. 후보가 사용하는 FP64 연산을 숨기지 않고 mixed라고 명시한다. 필요한 연산만 FP64로 남기는 것이 본래 목표에 부합한다.

## 8. 보정 FP32 힘 평가를 어디에 사용할 것인가

v2의 힘 평가 9.8배를 활용할 가능성은 열어두되, M2가 성공했다고 official force/line search/audit까지 FP32로 바꾸지 않는다. 그 부분은 최종 정확도를 결정한다.

필수 M2 실험에서 FP32 힘 경로는 low geometry 구성·진단 등에 사용할 수 있지만, nonlinear b와 trial acceptance는 FP64 원래 평가를 사용한다. 매 outer residual/line-search에서 필요한 FP64 평가를 포함한 비용이 결과다.

**조건부 PRED32:** M2가 원래 기준을 통과하고도 FP64 힘/line search가 W1 총시간의 큰 부분으로 남을 때에만, 최대 2회의 FP32 Newton-like predictor를 별도 임시 버퍼에서 실행해 원래 timestep의 endpoint acceleration 초기 추정값을 만드는 후보를 허용한다.

- 물리 시작 상태·a0·held force·dt·외력 시각은 변경하지 않는다. 임시 작업은 추가 시간 적분이 아니다.
- 이미 있는 corrected force/HVP를 재사용하고 새 nonlinear 모델/전처리기를 만들지 않는다.
- 임시 결과는 그 자체로 승인·commit하지 않는다. 그 결과에서 원래 FP64/Newmark solver가 원래 force/correction/line-search/갱신/에너지 기준을 모두 만족해야 한다.
- 실패/큰 residual 증가/비유한 상태면 원래 초기 추정값으로 돌아가며 임시 비용도 포함한다.
- 원래 basin과 다른 해로 갈 가능성을 배제했다고 주장하지 않는다. 회귀/궤적 비교를 함께 한다.
- 이는 별도 predictor policy 실험이므로 기본 M1/M2 결과와 분리한다. W1 profile에서 이득 여지가 작으면 `deferred_low_expected_benefit`로 남긴다.

“FP32-only 최종 판정”을 이번 성공 조건으로 강제하지 않는다. 순수 FP32 single-step 진단은 필요할 때만 1개 표본으로 수행하고 원래 검산 실패를 완화 기준으로 통과시키지 않는다.

## 9. frozen linear에서 실제 데이터 생성 구간까지

### 9.1 단계별 gate

- G0: 코드/정밀도/입력/참 연산자/스케일 역변환 검증.
- G1: L0/L1/L2에서 같은 원래 참 잔차 target까지 도달. iteration/time/fallback 전부 기록.
- G2: C0와 W1 3프레임에서 원래 solver+independent audit 통과. 일반 wall-clock 비교.
- G3: 유망 mixed 후보 하나와 FP64 reference만 **동일 W1 checkpoint에서 이어지는 원본 forcing 30프레임** 비교. 이전 3프레임이 이 30프레임의 시작 부분이 되게 한다. horizon은 성능·안정성 선별용 제안값이며 장기 보증이 아니다.

원본 forcing이나 대응 checkpoint가 없으면 임의 바람을 만들지 않는다. baseline으로 동일 history를 따라 준비할 수 있으면 준비비용을 별도 기록하고, 불가하면 G3를 `not_measured`로 남긴다. 바람 정지·감쇠 구간은 기존 궤적에 있을 때 추가 대조로만 사용한다.

G1에서 실패한 후보에 G3를 무조건 실행하지 않는다. 성공하지 못해도 matrix·오차·수렴곡선·원인분리 결과는 산출물이다. 진단만 하고 종료하기 전에 M1/M2 최소 L1 실행까지는 가능한 범위에서 실제로 시도한다.

### 9.2 정확도와 label 판정

매 단계의 공식 force residual, correction, Newmark update, energy ledger, geometry/status를 **동결 FP64 독립 auditor**로 확인한다. candidate의 audit dtype/수식을 함께 바꾸지 않는다. 종료 프레임만 통과하고 중간 위반을 숨기지 않는다.

각 동일 저장 시각에서 u/v/a가 있으면 해당 상태와 내력·에너지·원래 ledger 차이를 함께 남긴다. a를 u 차분으로 새로 계산해 참값처럼 쓰지 말고 저장/원래 evaluator를 사용한다.

hi/lo 상태 차이는

\[
\Delta u=(u_{hi}^{cand}-u_{hi}^{ref})+(u_{lo}^{cand}-u_{lo}^{ref})
\]

로, 먼저 pair를 단일 FP64로 collapse하지 않고 비교한다. 분석 host가 실제 extended precision을 지원하는지 확인하고 없으면 error-free transform/다중 정밀도 대조를 쓴다. v도 같다.

`official_audit_status`, `linear_true_residual_status`, `trajectory_regression_status`, `cross_device_status`, `performance_status`, `production_status`를 별도 필드로 둔다. 회귀 예산이 없으면 미정 상태를 유지하고 임의 완화하지 않는다.

### 9.3 반복·시간 비교

후보 탐색은 작은 반복 수로 하고, 최종 비교는 baseline A와 선택 후보 B를 `AB,BA,AB,BA,AB,BA`의 6쌍으로 실행한다. 각 실행마다 동일 checkpoint/factor 준비 조건을 복원하고 warmup·초기화·Graph capture 비용을 분리한다. 30프레임 gate는 우선 3쌍으로 선별 가능하며 반복 수를 명시한다.

\[
S=\operatorname{median}T_{ref}/\operatorname{median}T_{cand},\quad
D=100(1-\operatorname{median}T_{cand}/\operatorname{median}T_{ref}).
\]

기본 시간은 solver+audit를 보고하고, 생산 저장까지 포함한 validated generation 시간도 별도로 보고한다. 실패/재시도/fallback을 제거한 선택적 평균을 주 결과로 쓰지 않는다. 모든 raw·median/MAD/min/max·paired ratio를 남긴다. 프로파일러 시간과 일반 시간을 섞지 않는다.

## 10. 별도 FP64 추가 실험 — 빠르고 강한 대조군도 만든다

FP32를 유리하게 보이게 하려고 비효율적인 FP64 기준만 남겨 두지 않는다. 원본 R64와 개선 R64-best를 모두 보존하고 mixed를 양쪽과 비교한다. 단, 아래 모든 가지를 강제로 수행하지 않고 P0 비용에 따라 우선순위를 정한다.

### F64-P: 같은 전처리기 계열의 freshness/재구축 비용 — 필수 고정 선형 대조

L1에서 같은 A64,b64를 고정한 채 다음을 비교한다.

- 실제 사용된 P64(원래 generation/age).
- 동일 assembly 알고리즘으로 현재 상태에서 새로 구성한 fresh P64.

A64를 P64로 대체하지 않는다. 행렬 근사 자체와 precision 효과를 구분한다. fresh 조립/분해 비용 포함 여부를 분리해 기록한다.

\[
T_{old}=T_{apply+Krylov,old},\quad
T_{fresh}=T_{assemble}+T_{factor}+T_{apply+Krylov,fresh}.
\]

고정 선형계에서 fresh P의 이득이 보이면 실제 W1에 대해서만 원래 rebuild 정책과 `rebuild_every` 한 단계 감소(예: 현재64→32, 실제 단위는 Newton 호출 카운터인지 source 확인) 후보 하나를 비교한다. 최종 선형/비선형 target은 유지한다. 매 Newton 무조건 rebuild는 비용 상한 대조로 고정 선형에서만 우선 사용한다.

같은 freshness를 M1/M2에도 적용해 비교한다. FP32 후보만 fresh P를 쓰고 FP64는 오래된 P를 쓰는 비교를 precision 가속이라고 부르지 않는다. 새 AMG/ILU/분할 전처리기/CG 전환은 이번 범위 밖이다.

### F64-H: 실제 HVP 커널 launch 비교 — HVP가 주요 비용일 때

기존 force block 최적화가 HVP까지 적용됐다고 가정하지 않는다. 실제 HVP kernel의 launch/grid/register와 연산당 비용을 확인한 뒤, 상위 HVP 커널 1~2개에 한해 block=32/64/128/256 고정 작업량 비교를 한다. 수식/dtype/입력/호출 수는 동일하게 유지한다.

유망 후보 하나만 W1에 통합한다. force·HVP·linear 변경을 동시에 하지 않는다. 원래 assembled/operator outputs와 기존 audit를 확인한다. 작은 블록이 반드시 빠르다고 가정하지 않는다.

### F64-R: 이미 존재하는 재사용이 실제로 활성화됐는지 확인

`resident_preconditioner_reuse.py`, `resident_accepted_evaluation.py`, `resident_parallel_reductions.py`의 적용 여부를 확인한다. 이미 활성화돼 있으면 같은 기능을 재구현하거나 새 성과로 보고하지 않는다. 아직 남아 있는 동일 상태의 중복 A/P/force 평가가 profile에서 확인되는 경우에만 해당 중복 하나를 제거하고 stale state/cache invalidation을 검사한다.

직교화가 지배적이라면 finite32 최적화와 분리하여 restart 후보 하나를 FP64에서 비교할 수 있다. 참 잔차·전체 iteration cap 의미를 유지하고 재시작 수/메모리/재직교화 비용을 함께 기록한다. 큰 새 Krylov 계열 도입은 하지 않는다.

### F64-FMA: FP64 보상곱 단축 — 후순위 조건부

W1 profile에서 보상 기하/힘 비용이 여전히 크고 위 실험 다음으로 이득 여지가 있을 때만 `two_product`를 별도 후보에서 교체한다.

\[
p=\operatorname{RN}_{64}(ab),\qquad e=\operatorname{FMA}_{64}(a,b,-p).
\]

필요 연산은 명시적인 FP64 RN multiply와 명시적 FP64 FMA다. CUDA의 `__dmul_rn`/`__fma_rn` 의미를 확인한다. `p=a*b; e=a*b-p`로 쓰면 안 된다. 전체 `fuse_fp`/`fast_math`를 켜지 않고 나머지 TwoSum 등의 연산 순서를 보존한다.

유한값·overflow/underflow·subnormal 조건에서 무조건 error-free라고 선언하지 않는다. 실제 입력 범위와 경계값에서 high-precision reference로 (p,e)의 합/잔여값을 검사하고, 표현 불가 범위는 기존 경로로 보내거나 명시적으로 실패한다. primitive → fixed force/HVP → W1/audit 순서로 검증한다. baseline 독립 auditor는 원래 구현을 유지한다.

FMA 지정을 Warp/현재 빌드에서 안전하게 구현하기 어려우면 이 후보만 보류하고 M1/M2 결과를 완료한다. 새 툴체인 설치나 라이브러리 업그레이드를 이유로 작업 범위를 늘리지 않는다.

## 11. 두 PC 운영

공통 source·수식·최종 기준을 유지하되 GPU별 launch/precision policy는 다르게 선택할 수 있다. 5070에서 실패한 정책을 1080 Ti에 강제하지 않고 반대도 마찬가지다.

5070에서 L1 중심 개발을 먼저 완성하고, 유망 M1/M2와 R64만 1080 Ti에서 같은 snapshot·W1로 검증한다. 1080 Ti에서 mixed가 느리면 R64 유지도 정상 결과다. PC별 CPU/toolchain/backend/드라이버/Graph 기능/telemetry를 기록한다. 기존 cuDSS·Warp 환경을 보존하고 지원되지 않는 profiler를 강제하지 않는다.

동일 소스·원본 입력과 실제 전처리 배열을 구분해 hash를 남긴다. 교차 PC 비교는 같은 자유 DOF mapping·물리 시각·입력에 대해 수행한다. bitwise equality를 새 필수 기준으로 만들지 않는다. 승인된 교차 예산이 없으면 수치와 미판정을 함께 기록한다.

원격 접근이 기존에 승인된 경우에만 두 번째 PC를 실행한다. 접근 불가 시 현재 PC의 결과를 완료하고 다른 PC에서 실행할 정확한 명령과 collect 방법을 제공한다.

## 12. 최소 산출물과 측정 계약

v2를 덮어쓰지 않고 별도 `teacher_precision_v3/`에 저장한다. 루트 보고서에 최신 canonical 결과 경로를 명시한다.

1. `report.md`: 가설·선택 경로·성공/실패·다음 우선순위. FPS 대신 검산 포함 시간과 성공한 물리 길이를 함께 보고.
2. `acceptance_contract.json`, `precision_map.json`, `environment.json`: 기준/source/unit/norm과 연산별 실제 dtype.
3. `linear_systems/manifest.json`, 재현 NPZ/CSR: L0/L1/L2의 의미·인덱스·A/P distinction·hash·target.
4. `phase_times.csv`: GPU/case/role/parent/inclusive-or-exclusive/calls/time/profile-mode.
5. `operator_errors.csv`: system/variant/state-cast/direction/mode/role/Linf/L2/RMS/relative/status.
6. `linear_iterations.csv`: system/method/P_generation/age/inner_iter/correction_index/true_residual/target/rho/inner_estimated_residual/inner_true_residual/reason.
7. `linear_summary.csv`: setup/cast/scale/build/factor/A32/A64/P32/P64/reduction/inner/correction/fallback/total, original-target status.
8. `run_summary.csv`, `step_stats.csv`, `audits/`, matched snapshots: 실제 구간의 계산·검산·저장 시간, Newton/GMRES/IR/fallback/retry, flags.
9. `fp64_ablation.csv`, `precision_usage.csv`: 원본 R64·개선 R64·mixed 비교, 호출 수 및 FP32/FP64/mixed kernel 시간. 시간 비중을 FLOP 비중으로 부르지 않음.
10. `changes.diff`, `commands.txt`, source hashes, 원시 실행/telemetry/profiler CSV·텍스트, `failure_log.jsonl`.

C회 호출을 한 Graph에 capture한 fixed-work 측정을 유지한다. reset/zero/assembly/status 검사를 필요한 만큼 포함하며 alias·overwrite 때문에 실제 작업이 사라지지 않게 한다. 초기화/컴파일/capture·변환 비용은 분리하되 실제 per-Newton 변환 비용은 총시간에서 빼지 않는다.

새로운 mixed 경로가 eager이고 reference는 Graph인 경우 서로 표시한다. eager에서 정확도 확인 후 기존 구조를 활용해 Graph parity를 맞춘다. profiler 지원을 위해 생산 경로의 연산을 제거하지 않는다.

`NaN`과 `Infinity`를 값 검사 없이 `finite`로 표기하지 않는다. 필수 비교가 0개인 빈 결과를 all-pass로 처리하지 않는다. 모든 결과 숫자는 실제 측정으로만 채운다.

## 13. 실험 폭 제한과 종료 조건

기본 실행 순서: P0 → P1 → L1에서 M1/M2 및 F64-P → L0/L2 → 유망 후보의 C0/W1 → 유망 후보 하나의 30프레임. FP64-H/R/FMA는 profile이 정당화할 때 순차 실행한다.

필수 후보는 R64, M1, M2다. 주원인 분리용 구제 후보는 우선 1개, FP64 end-to-end 후보도 우선 1개만 선택한다. profile cache 체계를 다시 설계하거나 온갖 허용오차/블록/재구축 조합을 sweep하지 않는다.

같은 원인의 코드 수정·재실행이 2회 연속 실패하면 그 가지를 중단하고 로그/진단을 남긴다. 단순 numerical nonconvergence는 재현 가능한 실험 결과이지 무조건 코드 버그가 아니다. 실패한 후보와 아직 구현하지 못한 후보를 구분한다.

최종 보고서 첫머리에 다음을 답한다.

- 기존 target/audit를 그대로 통과한 FP32 중심 경로가 있는가?
- 있는 경우 어느 연산을 FP32로 했고 어떤 FP64 anchor를 얼마나 호출했는가?
- fallback 없는 mixed 성공과 FP64 fallback 포함 성공은 각각 몇 개인가?
- 원래 R64와 개선 R64 대비 frozen-linear 및 solver+audit 총시간이 각각 얼마나 줄었는가?
- FP32를 더 확대하지 못한 원인은 상태/HVP/분해/직교화/범위/반복비용 중 무엇으로 분리됐는가?
- 5070과1080 Ti의 선택 정책은 무엇인가? 미측정/미판정/생산 보류는 무엇인가?

## 14. 참고 근거와 해석 제한

### 제출 자료/코드

입력 ZIP 내부 `cases/W1/runtime/code/wind3dgs/teacher/`의 위 명시된 모듈과 v2 review의 §§3~8을 근거로 계획했다. 정확한 line number는 현재 checkout마다 달라질 수 있으므로 함수명과 hash로 대조한다. 아래 기술 자료는 방법의 근거이지 현재 문제에서의 가속/수렴 보증이 아니다.

### 1차 기술 자료

- N. J. Higham, “What Is Iterative Refinement?”, 2023-03-13. 저정밀 solve와 고정밀 residual/correction의 역할 및 조건수 제약.
  https://nhigham.com/2023/03/13/what-is-iterative-refinement/
- E. Carson and N. J. Higham, “Accelerating the Solution of Linear Systems by Iterative Refinement in Three Precisions”, SIAM J. Sci. Comput., 2018, DOI 10.1137/17M1140819.
- PETSc 공식 KSPFGMRES 문서: fixed GMRES와 flexible/nonlinear preconditioning 구분.
  https://petsc.org/release/manualpages/KSP/KSPFGMRES/
- SUNDIALS/KINSOL 공식 Mathematical Considerations: inexact Newton, scaling, 선형 정확도와 비선형 수렴, stopping 의미.
  https://sundials.readthedocs.io/en/latest/kinsol/Mathematics_link.html
- NVIDIA CUDA Math API, FP64 intrinsics: explicit multiply/FMA 반올림 동작. 현재 환경 API/컴파일 결과는 별도 확인.
  https://docs.nvidia.com/cuda/cuda-math-api/cuda_math_api/group__CUDA__MATH__INTRINSIC__DOUBLE.html
- NVIDIA cuDSS data types: pivot 및 내부 설정은 installed 0.7.1의 헤더·runtime에 대응시켜 읽는다. 최신 기본값을 과거 환경에 전용하지 않는다.
  https://docs.nvidia.com/cuda/cudss/types.html

**문헌에 혼합 정밀도 알고리즘이 존재한다는 사실만으로 이 shell의 모든 상태에서 원래 기준을 통과한다고 결론 내리지 않는다. 통과 여부와 경제성은 위 실험으로 판정한다.**

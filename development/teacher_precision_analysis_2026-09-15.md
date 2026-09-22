# Mesh teacher GPU/precision 성능 분석

분석일: 2026-09-15  
입력: `analysis-context.zip`  
입력 ZIP SHA-256: `7b01a8c1c4a571e2ed802a8c35396da20d4fb9132f8f971dc4d452d9d2491083`

## 0. 판정

현재 대표 구간의 주요 병목은 **FP64 산술 + 상위 힘 평가 커널의 불충분한 GPU 병렬 활용**이다. 특히 상위 edge/volume 커널의 표본에서 활성 SM의 FP64 파이프라인 사용률은 약 83%인 반면, 전체 경과 시간 기준 평균은 각각 15%, 47%이다. 이것은 서로 모순되지 않는다. 일부 SM만 오래 일하거나 마지막 소수 블록을 기다리는 상태다.

따라서 FP64가 문제라는 가설에는 구체적인 근거가 생겼다. 그러나 **전체 FP32 전환만이 해결책이라는 결론은 아니다.** 우선 현재 정밀도/물리식/허용오차를 유지하고 상위 두 커널의 block_dim을 시험하는 것이 비용·위험 대비 우선순위가 높다. 그다음 보상 기하 계산의 FP64 연산량을 줄이는 후보를 검증하고, 같은 커널의 기존 FP32 경로를 고정 작업량으로 비교한다.

FP32의 실제 가속률은 아직 측정되지 않았다. `fixed_work.csv`는 헤더뿐이고 `fixed_work_status.json`은 `not_measured`다. 이 보고서의 어떠한 가속 숫자도 FP32 실측 결과가 아니다.

분석 범위는 첨부 로그·CSV·코드의 정적 대조와 재집계다. GPU 솔버를 이 환경에서 새로 실행하거나 수정한 결과가 아니다.

## 1. 실제 실행 조건과 정상 시간

근거: `case.json`, `environment.json`, `run_summary.csv`, `raw/ordinary.csv`, `attempt_stats.json`.

- RTX 5070, 노드 1,813개, P3 삼각형 384개.
- 한 프레임 = 1/60초, 기본 내부 스텝 64개, dt=1/3840초.
- FP64 hi/lo 상태, FP64 힘/HVP/선형 풀이, FP64 hi/lo 독립 검산.
- backend 이름은 `newmark_gauss_retry`이지만 이번 실제 구간은 모두 base Newmark다. Gauss 재시도는 0이다.
- 일반 실행 3회 모두 64스텝 완료, 검산 통과. 프레임당 GMRES 총 192회, 행렬 rebuild 카운트 2회.
- 초기 조건은 preload checkpoint이며 바람 응답 시작 frame 0이다. 강한 바람·큰 변형·어려운 수렴 구간 전체를 대표한다고 볼 수 없다.

| 항목 | 실행 0 | 실행 1 | 실행 2 |
|---|---:|---:|---:|
| process wall (s) | 36.422823 | 9.945893 | 9.734967 |
| preprocess (s) | 0.458856 | 0.449440 | 0.459577 |
| setup (s) | 29.723498 | 4.641358 | 4.601394 |
| compute + audit (s) | 1.547272 | 1.549836 | 1.549797 |
| 나머지 미분해 시간 (s) | 4.693197 | 3.305260 | 3.124200 |

`compute_audit_s` 평균은 **1.548968초**, 중앙값은 **1.549797초**다. 최대-최소 폭은 평균의 약 **0.166%**로 작다. 마지막 두 프로세스 전체 시간 평균은 약 9.840초이지만, 이를 GPU 계산 1프레임 비용으로 쓰면 안 된다.

나머지 시간은 `process_wall - preprocess - setup - compute_audit`로 계산한 값일 뿐이며, 전부 CPU 병목·파일 저장으로 재명명할 근거는 없다. import, 동결 파일 해시 검사, 런타임 준비, 종료, 저장 등이 포함될 수 있다. 저장 시간은 따로 측정되지 않았다.

동일한 물리/반복 비용이 계속된다는 가정에서 1.548968초/프레임은 약 0.646 output frame/s, 실시간 대비 약 92.94배다. 이는 초기 비용과 별도 저장을 제외한 국소 환산이며 긴 궤적 성능 예측이 아니다.

### 1.1 초기 준비와 본 계산은 다른 최적화 대상

`source_excerpt/resident_newmark_gauss_retry.py:11-22`는 실제 재시도가 없어도 Gauss solver/audit를 미리 구성한다. 이는 setup 최적화 후보지만 **이번 측정의 본 계산 약 1.55초가 Gauss 때문이라는 뜻은 아니다.** 장기 생산에서는 초기화 재사용으로 상각할 수 있으며, 진단용 1프레임 새 프로세스 비용과 혼동하지 않는다.

## 2. Nsight Systems 커널 시간 재집계

근거: `nsys/cuda_gpu_kern_sum_cuda_gpu_kern_sum.csv`.

모든 커널 duration 합: **1.512339759초**, 커널 이름 108종, 호출 68,529개.

| 그룹 | 커널 시간 합 (ms) | 커널 시간 합 내 비율 |
|---|---:|---:|
| hi/lo 기하 + 힘/에너지 핵심 edge/volume | 755.310500 | 49.9432% |
| cuDSS 전진/후진 삼각 풀이 | 369.957380 | 24.4626% |
| 요소 HVP 핵심 edge/volume | 165.510126 | 10.9440% |
| 힘/HVP 조립 및 gather | 97.257280 | 6.4309% |
| cuDSS 수치 분해 계열 | 12.058322 | 0.7973% |
| 기타 커널 | 112.246151 | 7.4220% |

분류 규칙:

- 힘 핵심: `edge_kernel_*`, `volume_kernel_*`.
- HVP 핵심: `edge_hvp_kernel_*`, `volume_hvp_kernel_*`.
- 삼각 풀이: 이름에 `cudss::`와 `fwd_ker`/`bwd_ker` 포함.
- 분해 계열: `cudss::`와 `factorize`/`independent_ker` 포함.
- 조립/gather: `assemble_batch_*`, `assemble_hvp_*`, `gather_batch_*`, `gather_hvp_*`.

상위 단일 커널:

| 이름 | 합계 (ms) | 호출 수 |
|---|---:|---:|
| edge_kernel_1ec56708 | 415.586768 | 642 |
| volume_kernel_062a28b3 | 339.723732 | 321 |
| cuDSS bwd_ker (상위 variant) | 239.896482 | 896 |
| cuDSS fwd_ker (상위 variant) | 125.539595 | 448 |
| edge_hvp_kernel_31935e5c | 94.404521 | 1,724 |
| volume_hvp_kernel_4d4cec74 | 71.105605 | 862 |

**분모 주의:** 위 백분율은 프로파일 실행에서의 커널 duration 합을 분모로 사용한다. 일반 실행 wall time의 정확한 시간 분해가 아니며, GPU utilization도 아니다. 겹친 커널, 호스트 대기, 프로파일 오버헤드를 포함한 wall time 해석에는 원시 timeline이 필요하다.

해석: 힘 핵심 평가가 최우선이다. HVP만 가속하거나 cuDSS 분해만 가속하는 것은 이 구간의 절반을 차지하는 작업을 그대로 둔다. 특히 분해 계열은 0.8%라서 factorization-only 최적화의 본 계산 영향은 작다. 삼각 풀이 24.5%는 별도의 중요한 대상이다.

## 3. NCU: FP64와 병렬 활용이 동시에 제한한다

근거: `ncu/ncu_top1.csv`, `ncu/ncu_top2.csv`, `ncu/ncu_top3.csv`. 각 파일은 헤더/단위/단일 측정행이며 모든 호출의 통계가 아니다.

| 지표 | edge | volume | cuDSS bwd 표본 |
|---|---:|---:|---:|
| grid blocks | 9 | 54 | 736 |
| threads/block | 256 | 256 | 32 |
| registers/thread | 254 | 163 | 61 |
| allocated registers/thread | 256 | 168 | 64 |
| register 제한 blocks/SM | 1 | 1 | 32 |
| achieved occupancy (active 기준) | 16.001% | 16.655% | 21.831% |
| FP64 pipeline (active 기준) | 83.177% | 83.829% | 4.692% |
| FP64 pipeline (elapsed 기준) | 15.023% | 47.164% | 4.453% |
| DRAM bytes/s | 2.589 GB/s | 4.456 GB/s | 6.031 GB/s |

NCU device metric의 SM 수는 **48**, SM당 레지스터 수는 **65,536개**다. 여기서 레지스터 개수는 하드웨어 32비트 레지스터 단위다.

### 3.1 active와 elapsed의 차이

`pct_of_peak_sustained_active`는 해당 하드웨어 단위가 active인 cycle을, `pct_of_peak_sustained_elapsed`는 elapsed cycle을 분모로 쓴다. 따라서 active 83%를 GPU 전체가 83% 활용된다는 뜻으로 읽으면 틀린다.

상위 edge는 9블록뿐이다. 한 커널의 9블록이 동시에 사용할 수 있는 SM은 최대 9개이므로, 다른 커널과의 동시 실행을 별도로 논하지 않는 이 표본에서는 최대 9/48=18.75%의 SM에만 일을 나눌 수 있다.

측정 cycle에서 계산하면:

\[
f_{\rm active,edge}=\frac{620894.375}{3437645.416667}\approx0.180616.
\]

FP64 두 지표의 비율도 거의 같다:

\[
\frac{15.023132}{83.177110}\approx0.180616.
\]

즉, 일하는 SM의 FP64는 바쁘지만 SM 대부분이 해당 커널에 참여하지 않는 해석과 일치한다.

volume은 54블록이며 레지스터 제한 때문에 SM당 1블록이 상주할 수 있다. 모든 블록의 실행 시간이 같고 다른 제한이 없다는 단순 모델에서는 첫 wave가 48블록, 마지막 wave가 6블록이다. 평균 사용 비율은:

\[
f_{\rm ideal,volume}=\frac{54}{48\times\lceil54/48\rceil}=\frac{54}{96}=0.5625.
\]

실측 cycle 비율:

\[
f_{\rm measured,volume}=\frac{1910520.0625}{3395733.791667}\approx0.562624.
\]

이 일치는 **tail effect가 중요한 원인이라는 강한 정황**이다. 정확한 두 wave의 시작/종료를 timeline으로 재구성한 결과는 아니므로 단순 모델의 해석으로 제한한다.

### 3.2 레지스터의 의미

edge의 할당량: 256 registers/thread × 256 threads/block = 65,536 registers/block.

volume의 할당량: 168 × 256 = 43,008 registers/block. 두 블록이면 86,016으로 SM당 예산을 넘는다.

두 경우 모두 NCU의 register-limit 1 block/SM과 일치한다. 다만 높은 레지스터 사용량만으로 spill 발생을 확정할 수 없다. 이번 raw CSV에는 그 주장을 뒷받침할 상세 spill/stall 정보가 없다.

### 3.3 메모리 병목 해석

상위 force 표본은 DRAM 처리량이 낮고 FP64 active 지표가 높다. 따라서 이 표본들을 **DRAM 대역폭 포화**라고 분류할 근거는 없다. 이것은 메모리 latency/의존성이 전혀 없다는 뜻은 아니다.

cuDSS bwd 표본은 FP64 active 4.692%, DRAM 6.031GB/s로 두 자원의 peak 처리량을 밀어붙이는 모습이 아니다. 의존성, 스케줄링, 작은 sparse 문제의 병렬성, 메모리 latency 등의 가능성은 있지만 현재 자료로 그중 하나를 확정할 수 없다. 특히 forward solve와 다른 grid의 backward 호출에는 이 수치를 그대로 일반화하지 않는다.

## 4. 코드에서 확인되는 비싼 FP64 작업

아래 행 번호는 원본 저장소 전체가 아닌 **첨부 source_excerpt 파일의 행 번호**다.

| 근거 | 확인 내용 |
|---|---|
| p3_shell_warp_precision_kernels.py:14 | fast_math=False, fuse_fp=False |
| 동 파일:18-47 | two_sum, 분할곱 two_product, pair_add/pair_scale |
| 동 파일:85-118 | hi/lo 기하 계산 |
| 동 파일:95-104 | 10노드 반복에서 f0/f1/h0/h1/h2를 보상 누적 |
| 동 파일:130-143 | volume의 load_geometry → 구성식/gradient/diagnostics |
| 동 파일:158-173 | edge의 양쪽 기하와 경계 gradient 계산 |
| p3_shell_resident.py:23-47 | force 평가의 실제 호출/조립 경로 |
| 동 파일:31-32,39-42 | 상위 volume/edge launch에 block_dim을 명시하지 않음 |
| p3_shell_cudss.py:86-89 | 내부 IR 및 hybrid 설정을 명시적으로 끔 |
| 동 파일:133-139 | 검증된 cuDSS 0.7.1 버전 요구 |
| 동 파일:165-168 | 전처리 적용 시 solve_graph 실행 |

이번 상위 커널의 기하는 단일 FP64 합산이 아니라 FP64 hi/lo 보상 연산을 반복한다. 이 때문에 일반적인 단일 FP64 요소 평가보다 산술·레지스터 비용이 커질 수 있다. 구성식/HVP 전체가 double-double이라는 의미는 아니다.

**이전 일반적 가설의 정정:** cuDSS 내부 IR과 외부 GMRES의 중복을 우선 의심할 이유는 이번 source에서 약해졌다. 코드가 IR=0을 의도적으로 설정하고 버전을 0.7.1로 고정한다. 최신 cuDSS 0.8 옵션을 설치 버전 확인 없이 적용하면 안 된다.

### 4.1 우선 최적화: 계산식 그대로 block_dim sweep

상위 두 커널만 block_dim ∈ {32,64,128,256}으로 비교한다. 초기에는 edge와 volume을 각각 독립적으로 바꾸고, 최선 후보 조합을 따로 검증한다.

이유: 각 thread가 (e,q) 출력 칸을 기록하는 구조이고, 위 두 커널의 본문에는 cross-thread reduction이 없다. 따라서 이 두 launch의 block 크기만 바꾸는 것은 FP32 전환이나 물리식 변경보다 수치적 위험이 낮은 시험이다. 단, 최종 출력을 실제로 확인해야 하며 자동으로 bitwise 동일하다고 보증하지 않는다.

중요 조건:

- force/assembly/gather의 순서, 반복 정책, dt, 허용오차는 바꾸지 않는다.
- CUDA Graph는 후보 설정으로 새로 생성한다. 기존 graph를 재사용하면서 옵션만 바꾸었다고 기록하지 않는다.
- 경계 edge와 내부 edge를 구분해서 시간/실제 dim/출력 오차를 기록한다.
- 작은 block이 무조건 빠르다고 가정하지 않는다. occupancy, 사용 SM 수, warp 배치와 컴파일 레지스터 수는 다시 측정한다.
- 처음부터 maxrregcount를 낮추지 않는다. 강제 register cap은 spill을 늘릴 수 있어 별도 원인 분석이 필요하다.

### 4.2 다음 후보: FP64 FMA 기반 TwoProduct

기존 분할곱은 source의 split 상수 134217729.0을 사용하는 여러 곱/덧셈으로 오차를 복원한다. 명시적 FP64 FMA를 사용하면 후보 알고리즘은:

\[
p=\operatorname{RN}_{64}(ab),\qquad e=\operatorname{FMA}_{64}(a,b,-p).
\]

p는 반올림된 곱, e는 그 곱의 잔여 오차다. overflow가 없고 잔여 오차가 underflow로 소실되지 않는 등 error-free transform의 조건에서 p+e=ab를 실수 의미로 만족한다.

CUDA native 구현의 핵심은 다음과 같다:

```cpp
const double p = __dmul_rn(a, b);
const double e = __fma_rn(a, b, -p);
// return (p, e) to the existing hi/lo representation
```

이는 FP32 전환도 허용오차 완화도 아니다. FP64 보상곱 구현 후보 변경이다. Warp에서 명시적 intrinsic/native 경계를 어떻게 구현할지는 해당 설치 버전에서 확인해야 한다. 위 코드는 전체 Warp 패치가 아니다.

**모듈 전체 fuse_fp=True/fast_math=True 변경과는 다르다.** TwoSum 등 반올림 순서가 중요한 부분은 유지하고, TwoProduct의 필요한 FMA만 명시한다. 입력 물리 범위와 극단값의 단위시험, 높은 정밀도의 p+e 검증, 전체 기하/힘/HVP 출력 비교, 기존 독립 검산이 필요하다. 속도 이득과 레지스터 감소는 미측정이다.

### 4.3 FP32 시험의 올바른 경계

같은 저장 입력과 호출 수에서 최우선 두 커널의 기존 FP32 hi/lo 경로를 측정한다. 단일 FP32도 이미 있다면 구분해 측정한다. absent 경로를 무리하게 새로 만들지 않는다.

`load_geometry`, 법선/변형률, 구성식/gradient 중 어디를 FP32로 낮췄는지 기록한다. 힘 커널 전체를 FP32로 만드는 일은 방정식 평가 오차와 수렴을 바꿀 수 있으므로, 성능 microbenchmark와 생산용 채택 여부를 분리한다. 이후 수치적으로 안전한 geometry/constitutive 정밀도 분할을 고를 수 있지만, 자료 없이 어떤 경계가 정답이라고 단정하지 않는다.

고정 작업량 비교에서 확인할 것은 **가속 여지가 있는지**다. 이 결과가 있어야 FP32 수렴 개선에 추가 시간을 투입할 가치가 얼마나 되는지 알 수 있다.

## 5. 기존 혼합 정밀도 개선이 작을 수 있는 이유

현재 frame에서 cuDSS 삼각 풀이가 약 24.46%다. 다른 시간이 고정이면 이를 무한히 빠르게 해도 커널 duration 합 기준 최대 가속은:

\[
S_{\max}=\frac{1}{1-0.2446258374}\approx1.32385.
\]

두 배 빠른 경우는:

\[
S_2=\frac{1}{1-0.2446258374+0.2446258374/2}\approx1.13936.
\]

그 위에 FP64 보정·추가 Krylov 반복 등이 붙으면 이득이 작아질 수 있다. 따라서 앞선 6.8% 혼합 정밀도 개선과 모순되지 않는다. **다만 앞선 실험은 Gauss의 다른 국소 구간이므로 이 수치를 그 실험에 역으로 대입해 원인을 확정하면 안 된다.**

힘 핵심 두 커널의 합 p=0.499431755에 대해, 다른 모든 비용이 변하지 않는 모델:

\[
S(s)=\frac{1}{(1-p)+p/s},\quad
p=\frac{0.755310500}{1.512339759},\quad
s=\frac{T_{\rm old,force}}{T_{\rm new,force}}.
\]

| 두 커널 자체 가속 s (가정) | 커널 duration 합 모델의 가속 |
|---:|---:|
| 2 | 1.33283 |
| 4 | 1.59891 |
| 8 | 1.77621 |
| 무한대 | 1.99773 |

이 표는 FP32/FP64 실측 비교나 생산 wall-clock 예측이 아니다. 프로파일된 커널 시간의 정적 비중을 이용한 우선순위 계산이다. 2배 이상의 큰 전체 가속에는 여러 그룹을 개선하거나, 실제 데이터 생성의 반복량/검산 비용까지 줄이는 전략이 필요할 수 있다. 현 단계에서는 허용오차 변경을 성능 변경과 섞지 않는다.

## 6. 측정 해석의 한계와 수정사항

### 6.1 NCU conditional-graph 제한

세 NCU log 모두 다음 경고가 있다:

> Kernel nodes of a graph which can have conditional nodes are not supported for profiling.

환경의 ncu는 2025.2.1이다. 공식 릴리스 노트상 conditional graph의 node-level profiling 지원은 2025.4에 추가되었다. 따라서 이번 표본을 '솔버 모든 호출 중 첫 번째'라고 확정할 수 없다. 프로파일 가능한 호출만 수집되었을 수 있으며 audit 호출일 가능성도 현재 bundle에서 구분되지 않는다.

이 경고가 유효하게 수집된 CSV 전체를 무효화하지는 않는다. 하지만 **solver/audit, interior/boundary, 첫 호출/정상 반복 호출의 대표성을 확인해야 한다.** 가장 작은 보완은 실제 입력을 재사용한 graph 밖 isolated fixed-work harness다. 도구 업그레이드는 선택이며, 측정용 도구 변경과 solver/runtime 변경은 분리한다.

### 6.2 독립 검산 시간은 분리되지 않음

NVTX는 `teacher_measure` 한 개뿐이다. `compute_audit_s`도 solver+audit 합계다. `resident_newmark_retry.py:29-47`에는 solver 단계와 audit 단계가 순차적으로 있으므로 별도 NVTX 범위를 넣을 수 있다.

같은 edge/volume 이름이 solver와 audit에서 모두 호출되면 Systems 합계에 합쳐진다. 따라서 이번 49.94%를 전부 solver의 force 비용이라고 부르지 않는다. 다음 계측에서 `solver_step_batch`, `independent_audit`, `save_or_transfer`를 구분한다.

### 6.3 CPU 시간 중복 합산 금지

`cuda_api_sum`에는 cudaGraphLaunch 약 0.80149초, cuCtxSynchronize 약 0.79802초가 있다. API 내 wall time과 GPU kernel duration은 겹칠 수 있으므로 서로 더하지 않는다. 특히 synchronize는 GPU 완료를 기다리는 시간을 포함할 수 있다. 이 값만으로 Python/CPU가 전체 병목이라고 판정할 수 없다.

NVTX wall은 1.659024644초이고 GPU kernel duration 합은 1.512339759초다. 둘의 비율 약 91.16%는 GPU utilization이 아니다. 원본 timeline을 제공하지 않았으므로 정확한 GPU busy-time union과 gap 분석은 보류한다.

### 6.4 반복과 정확도

per-step Newton/GMRES 횟수 열은 비어 있다. 프레임 GMRES 합계 192를 64로 나누면 평균 3회/accepted step이지만 모든 step이 3회였다는 뜻은 아니다.

같은 초기 체크포인트의 일반 실행들에서 반복 합계와 성공 여부는 일치하지만 checks 배열은 bitwise 같지 않다. 이 보고서는 이름/단위가 없는 6개 checks 열을 힘[N] 등으로 임의 재명명하지 않는다. 다음 산출물에는 checks 열의 label, unit, scale/normalization 정의를 포함한다.

`training_eligible=false`는 이 bundle의 정책 표기다. 실제 기록의 audit pass와 구분한다. 이 자료만으로 teacher의 장기 정확성/학습 적합성이나 FP32 정확도 예산이 검증된 것은 아니다.

### 6.5 1080 Ti/5070 비교 범위

첨부에 1080 Ti preload 보고서는 있지만, 같은 wind checkpoint/동일 실행범위를 재생한 두 장치 비교표는 없다. 사용자가 앞서 관측한 1.2배를 이번 bundle만으로 재검증할 수는 없다. 현재 자료가 제공하는 직접 근거는 RTX 5070의 병목 진단이다.

## 7. Codex 다음 작업 명세

### 목적 및 변경 금지 범위

목적: **기존 검산을 유지하면서 force 핵심 커널의 GPU 활용을 개선하고 FP32 가속 잠재력을 실측한다.**

물리식, P3 요소, 구적 규칙, 재료, dt, Newton/GMRES 허용오차, line search, 검산 판정, 전처리 알고리즘은 변경하지 않는다. 하나의 baseline에 여러 변경을 동시에 적용하지 않는다.

### P0 — 측정 구간 분리 + FP64 block sweep

1. 기존 동일 checkpoint와 force 입력을 고정한다. 내부 edge, 경계 edge, volume을 구분한다.
2. solver batch / independent audit / 원시 상태 전송·저장을 NVTX로 분리한다. 일반 실행에는 필요한 최소 계측만 넣고 커널마다 synchronize하지 않는다.
3. 상위 volume/edge 커널만 FP64 hi/lo 유지 상태에서 block_dim=32,64,128,256을 각각 고정 호출 수로 측정한다. 워밍업과 초기화는 별도 기록한다.
4. 기존 Graph를 해당 설정으로 재생성한 1프레임 일반 실행도 3회 비교한다. 기존 검산, status, 출력 차이, 반복 합계, rebuild 횟수를 같이 기록한다.
5. 가장 빠른 검산 통과 설정을 선택하되, 1프레임 승리를 장기 성능으로 명명하지 않는다.

P0 산출물 CSV 열 예시:

```text
operation,role,edge_kind,precision,block_dim,grid_dim,repeat,calls,
gpu_total_ms,gpu_us_per_call,registers_per_thread,spill_status,
output_error_kind,output_abs_error,output_rel_error,finite,
compute_audit_s,audit_pass,gmres_total,matrix_rebuilds,status
```

role=solver/audit/isolated, edge_kind=interior/boundary/not_applicable. 없는 값은 빈칸+이유를 기록하고 추정값으로 채우지 않는다.

### P1 — 동일 입력의 기존 FP32 경로 비교

P0에서 선택한 블록 구성과 기존256 모두에서 가능한 기존 FP32 hi/lo 경로를 측정한다. 단일 FP32가 있다면 별도 행으로 기록한다. 힘 평가 하나의 전체 연산 경로를 비교하되 데이터 변환/복원 시간과 steady GPU 시간을 분리한다.

정확도는 단순 출력 norm만 보지 말고 geometry, assembled force, 필요시 HVP의 차이를 기록한다. 이 단계에서는 FP32 Newton 완주를 합격 조건으로 삼지 않는다. 생산용 채택은 원래 검산/추가 장기 구간으로 별도 판정한다.

### P2 — FP64 TwoProduct-FMA 후보

P0/P1과 독립된 variant로 기존 two_product만 명시적 FP64 FMA 구현으로 바꾸어 primitive→geometry→force→frame 순서로 검증한다. TwoSum/전체 fuse_fp/fast_math는 바꾸지 않는다. 큰가속을 사전 약속하지 않는다.

primitive 검증은 정확한 float64 입력을 MPFR 또는 exact rational로 해석해 잔여 오차를 확인한다. p+e를 FP64로 먼저 더한 뒤 '같다'고 판단하면 lo 정보가 사라지므로 그런 검증을 사용하지 않는다. underflow/overflow 조건과 처리 정책을 명시한다.

### 보류

새 multigrid/새 shell 모델/penalty 재설계/전체 solver 재작성/FP32 전체 완주/허용오차 대폭 완화는 이번 작업에서 제외한다. cuDSS 삼각 풀이의 stall 분석이나 batched 독립 궤적은 위 작업 후 실제 잔여 병목을 보고 판단한다.

## 8. 최종 권고

**FP64는 실제로 중요한 병목이다. 그러나 현재 5070은 그 제한된 FP64 자원조차 전체 SM에서 활용하지 못한다.** 가장 우선적인 조치는 상위 두 힘 평가 커널의 작은 block 시험이다. 다음은 비싼 보상 기하 연산의 FP64 유지 최적화와 실제 FP32 고정 작업량 비교다.

이 순서라면 정확도 문제를 새로 만들지 않는 최적화부터 시도하면서, FP32 수렴 개선에 투자할 성능상 근거도 얻을 수 있다.

## 외부 기술 근거

측정 숫자와 소스 판단의 1차 근거는 위 첨부 파일이다. 아래 문서는 metric/수치 primitive의 의미를 확인하는 데 사용했다.

- NVIDIA Nsight Compute Profiling Guide, active/elapsed metric definitions: https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html
- NVIDIA CUDA C++ Best Practices Guide, register pressure/occupancy/execution configuration: https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html
- NVIDIA Nsight Compute Release Notes, Updates in 2025.4 (conditional graph node profiling): https://docs.nvidia.com/nsight-compute/ReleaseNotes/index.html
- NVIDIA CUDA Math API, double-precision __dmul_rn / __fma_rn: https://docs.nvidia.com/cuda/archive/13.0.3/cuda-math-api/cuda_math_api/group__CUDA__MATH__INTRINSIC__DOUBLE.html
- NVIDIA Floating Point and IEEE 754, explicit intrinsics/FMA behavior: https://docs.nvidia.com/cuda/floating-point/index.html

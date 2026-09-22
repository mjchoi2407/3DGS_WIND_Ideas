# 두 GPU Phase 1 결과 검토 — 2026-09-16

## 0. 판정 요약

**RTX 5070의 FP64 launch 튜닝은 이 시험 구간에서 유효한 성능 개선을 보였다. GTX 1080 Ti의 개선 여부는 측정 변동 때문에 미판정이다. FP32 가속 여지는 확인되지만, 측정 경로가 legacy이며 정확도 보정 경로의 가속률은 아직 확인되지 않았다.**

- RTX 5070: FP64 hi/lo, 기존 독립 검산 유지. `(volume, interior_edge, boundary_edge)=(32,32,256)` 후보에서 solver+audit 중앙값 **1.562877 → 1.226506 s**, **1.274251×**, **21.5226% 시간 감소**.
- GTX 1080 Ti: `(64,64,64)` 후보. 원시 중앙값 비율은 1.004871×지만, baseline 범위가 1.929–7.463 s여서 개선의 증거로 사용할 수 없다. 유지할 생산 기본값은 baseline.
- 두 PC 모두 기존 검산을 통과했으나, 자동 채택 조건에 `linf == 0`이 들어 있어 프로파일은 둘 다 `baseline_retained`이다. 실제 baseline 자체도 반복 실행 간 완전히 일치하지 않는다.
- 실제 측정한 프레임의 입력 바람은 `forcing.npz['wind'][0] == [0,0,0]`이다. 중력과 비영 초기 속도가 있는 preload 이후 한 프레임이지, 비영 외부 바람 응답 검증은 아니다.
- FP32 경로는 `specialize(..., diagnostic=False)`의 기본값 `strain_formula='legacy'`를 사용했다. 이미 구현된 stable strain/geometry/metric 보정 경로는 이번 fixed-work에 사용되지 않았다.

## 1. 검토 범위와 증거

입력:

- `dual_gpu_gtx1080ti_v1.zip`
- `dual_gpu_rtx5070_v1.zip`
- 기존 `codex_teacher_precision_dual_gpu_request_2026-09-15.md`

검토자는 CSV·NPZ를 재계산하고 동봉 소스를 정적으로 대조했다. **GPU solver를 재실행하지 않았다.** `force_launch_profile.py`의 순수 CPU 설정 선택 함수만 합성 fixture로 단위시험했다. 이 시험은 실제 GPU 프로파일 채택 또는 teacher 검증이 아니다.

각 ZIP의 `experiment_manifest.json`에 기록된 파일을 SHA-256으로 검증했다. GTX 1080 Ti 712개, RTX 5070 716개 항목 모두 일치했다. 두 ZIP의 `input_source_hashes.json`은 동일하다. `runtime/code`, `fp32_hilo/code`, `fp32/code`는 각각 192개 파일을 대조했으며 두 ZIP 사이에 내용 차이가 없다.

단, 런타임에 재생성한 구적/미분 계수 지문은 서로 다르다. 원본 입력·소스 일치와 실제 생성된 모든 배열의 비트 일치는 구분해야 한다.

이 문서의 `W`는 각 ZIP의 `workers/<gpu>/`, `R`은 `runtime/code/wind3dgs/`를 뜻한다. 줄 번호는 ZIP에 동봉된 소스 기준이다.

## 2. 전체 시간 재집계

근거: `W/run_summary.csv` 2–10행. baseline은 `baseline_r0..2`와 `baseline_post_r0..2` 6회, 후보는 `candidate_r0..2` 3회다. GPU 프로파일러 실행은 포함하지 않았다.

| GPU | 설정 | n | solver+audit 중앙값 (s) | MAD (s) | min–max (s) |
|---|---|---:|---:|---:|---:|
| GTX 1080 Ti | baseline 256/256/256 | 6 | 2.289891 | 0.298802 | 1.929227–7.462665 |
| GTX 1080 Ti | candidate 64/64/64 | 3 | 2.278791 | 0.086303 | 2.192487–6.349286 |
| RTX 5070 | baseline 256/256/256 | 6 | 1.562877 | 0.001459 | 1.561054–1.568985 |
| RTX 5070 | candidate 32/32/256 | 3 | 1.226506 | 0.000699 | 1.225575–1.227205 |

계산식:

\[
\widetilde T=\operatorname{median}_r T_r,\qquad
\operatorname{MAD}=\operatorname{median}_r|T_r-\widetilde T|,
\]
\[
S=\frac{\widetilde T_{\rm baseline}}{\widetilde T_{\rm candidate}},\qquad
r_T=100\left(1-\frac{\widetilde T_{\rm candidate}}{\widetilde T_{\rm baseline}}\right)\%.
\]

`1.274×`는 처리율 약 27.4% 증가이고, 시간 감소율은 약 21.5%다. 두 백분율은 다르다.

### 2.1 RTX 5070: 개선은 반복 측정에서 일관적이다

| 구간 | baseline 중앙값 (s) | candidate 중앙값 (s) | 가속률 | 시간 감소 |
|---|---:|---:|---:|---:|
| solver | 1.339229 | 1.071078 | 1.250356× | 20.0228% |
| independent audit | 0.222872 | 0.154790 | 1.439835× | 30.5476% |
| solver+audit 직접 측정 | 1.562877 | 1.226506 | 1.274251× | 21.5226% |

각 항의 중앙값을 합한 값은 전체의 중앙값과 정확히 같을 필요가 없다. 작은 제어 비용도 전체 시간에만 포함된다.

후보 바로 뒤 baseline과의 쌍별 시간 감소율은 21.5294%, 21.4710%, 21.7386%다. 따라서 첫 baseline의 준비 효과만으로 나타난 차이가 아니다.

양쪽 GPU의 9회 실행 모두 64개의 Newmark 내부 스텝을 완료했고, 한 실행당 GMRES 192회, rebuild 2회, retry 0회다. 시험이 더 적은 작업을 수행해서 빨라졌다는 정황은 없다. 이름이 `newmark_gauss_retry`여도 이번 실행은 Gauss로 전환되지 않았다.

이 비율은 초기화·JIT·최종 NPZ 저장 등을 포함한 모든 프로세스 비용의 가속률은 아니다. RTX 5070 `process_wall_s` 중앙값은 baseline 11.0371 s, candidate 10.7558 s이며, setup 영향이 커 21.5%를 그 전체에 적용하면 안 된다. 연속 데이터 생성의 최종 성과는 초기화 상각과 실제 label 저장을 포함해 별도로 평가해야 한다.

### 2.2 FP64 힘 커널 실행 구성의 실제 개선

RTX 5070 fixed-work에서 run별 3회 중앙값을 별도로 계산했다. `fp64_hilo_b256` 대 `fp64_hilo_selected` 비교다.

| 역할 | baseline (µs/call) | candidate (µs/call) | 비율 |
|---|---:|---:|---:|
| volume | 1165.725 | 693.611 | 1.68066× |
| interior edge | 1175.832 | 501.354 | 2.34531× |
| boundary edge | 252.274 | 252.389 | 약 1.00× |
| assembled_force | 2975.931 | 1830.051 | 1.62615× |

`assembled_force`는 앞의 세 역할을 포함하는 evaluate 전체이며, 이 네 행의 시간을 더하면 안 된다. 또한 이 표는 eager fixed-work 실행이다. 생산 Graph replay 커널 시간과 동일하다고 간주하지 않는다.

실제 Graph metadata에서 volume grid는 54 → 432 blocks, interior edge는 9 → 69 blocks로 바뀐다. boundary는 1 block이며 최종 후보에서 256을 유지했다. 확인 위치: `W/snapshots/{baseline_r0,candidate_r0}/result.json`의 `graph`, `launches`.

새 NCU 계측은 `Launching the target application failed.`로 실패했다. 따라서 이번 결과로 새 occupancy 또는 FP64 pipeline 포화율을 측정했다고 주장하면 안 된다. 성능 개선 자체는 wall-clock과 fixed-work에서 확인된다.

## 3. 자동 채택 보류는 수치 검산 실패가 아니다

**우선 수정할 판정 로직:** `R/evaluation/teacher_dual_gpu.py:263–284`.

실제 코드:

```python
exact &= comparison=='compared' and all(
    x['status']=='finite' and x['linf']==0
    for x in values if x['quantity']!='ledger_check_max_j'
)
adopted=bool(improved and audit_ok and exact and graph_ok)
```

최종 값의 완전 일치를 새 필수 조건으로 사용하고 있다. 기존 지시서 140행의 “bitwise equality는 관측 항목이지 새 필수 조건이 아니다”와 맞지 않는다. 엄밀히 `linf == 0`은 저장 바이트의 bitwise equality와 동일한 검사도 아니지만, 비교한 값의 정확한 동일성을 요구한다는 점에서 문제의 성격은 같다.

현재 결과는 다음을 분리해서 표현해야 한다.

- `performance`: RTX 5070 개선 확인, GTX 1080 Ti 미판정.
- `physical_audit`: 양쪽 기존 검사 통과.
- `exact_equality`: 실패.
- `numerical_regression`: baseline 자체 변동과 비교해야 하며, 공식 회귀 예산이 없으면 미정 상태를 보존.
- `production_promotion`: 아직 수행하지 않음.

### 3.1 baseline 자체도 정확히 같지 않다

`W/snapshots/*/snapshot_0001.npz`를 직접 재계산했다. free 노드는 1764개, 비교 성분은 5292개다. hi/lo를 단일 FP64로 합치지 않고 다음 차이를 extended precision으로 계산했다.

\[
d_{i,c}=(h^A_{i,c}-h^B_{i,c})+(\ell^A_{i,c}-\ell^B_{i,c}),
\]
\[
e_{\infty}=\max_{i\in F,c}|d_{i,c}|,\qquad
e_{\rm rms}=\sqrt{\frac{1}{3|F|}\sum_{i\in F}\sum_{c=1}^3 d_{i,c}^2}.
\]

힘과 에너지는 저장된 FP64 값을 extended precision으로 뺀다. 아래는 RTX 5070의 끝 시각 `1/60 s` 결과다.

| 양 | baseline 6회 상호 비교 최대 L∞ | baseline 6회 × candidate 3회 최대 L∞ |
|---|---:|---:|
| 변위 u (m) | 1.774446e-17 | 1.672567e-17 |
| 속도 v (m/s) | 5.732997e-13 | 5.587687e-13 |
| 힘 (N) | 4.661184e-13 | 4.954517e-13 |
| 탄성 에너지 (J) | 5.353248e-19 | 4.506215e-19 |
| 운동 에너지 (J) | 5.439255e-21 | 4.447776e-21 |

튜닝 차이는 baseline 반복 변동과 같은 규모다. 특히 변위·속도 차이는 baseline 간 최대 차이보다 작다. 힘의 최대 차이는 약 6.3% 크지만 동일한 1e-13 N 규모이며, 이 소수 표본으로 정확한 확률이나 통계적 동등성을 주장하지 않는다.

별도로 같은 상태의 FP64 fixed-work에서는 block 32/64/128/256 및 selected의 저장된 힘·기하 출력이 일치한다. 이것과 전체 궤적의 비결정적 차이를 구분해야 한다.

**판정:** 튜닝으로 의미 있는 수치 열화가 생겼다는 증거는 없다. 그렇다고 이 결과만으로 새 teacher 정확도 기준을 발명하거나 장기 검증 통과로 승격하지 않는다. baseline 재현 편차는 회귀의 진단 기준이며 물리 허용오차의 대체물이 아니다. 기존 공식 회귀 예산이 있으면 그것을 적용하고, 없으면 작은 차이의 원자료와 미정 상태를 보존하되 성능 성공까지 실패로 표현하지 않는다.

근거 추가: 18회 전체의 독립 검산 flags는 모두 0. 정규화 force residual/allowance 최대는 GTX 1080 Ti 약 0.006679, RTX 5070 약 0.006538로 1보다 작다. position update 및 energy ledger 재평가 항목도 기존 검사를 통과했다. 이 값은 장기 물리 정확도나 시간/공간 수렴의 증명은 아니다.

## 4. GTX 1080 Ti는 재측정 전 최적 block을 확정하지 않는다

근거: `W/run_summary.csv`, `W/fixed_work.csv`, `W/logs/nvidia_smi.log:9–10`.

후보 바로 뒤 baseline과 비교한 값:

| 반복 | baseline_post (s) | candidate (s) | 후보 시간 감소 |
|---|---:|---:|---:|
| 0 | 6.277320 | 6.349286 | -1.1465% |
| 1 | 2.271009 | 2.192487 | 3.4576% |
| 2 | 2.308773 | 2.278791 | 1.2986% |

“0.48% 개선”을 확정하는 것도, “1080 Ti는 최적화가 불가능”이라고 결론 내리는 것도 근거가 약하다.

또한 **같은 block=64의 같은 volume 역할**이 초기 `fp64_hilo_b64`에서는 2958.44 µs/call, 후반 `fp64_hilo_selected`에서는 5025.33 µs/call 중앙값이다. 동일 설정의 순위가 시간에 따라 흔들릴 여지가 크다. 최적화 후보 (64,64,64)를 버릴 필요는 없으나 확정 설정으로 등록할 수 없다.

nvidia-smi 단일 표본은 83°C, GPU-Util 97%, P0, 139W/250W를 기록했다. **이 한 표본만으로 thermal throttling 또는 다른 프로세스 간섭을 확정할 수 없다.** GPU clock, memory clock, 온도·전력·제한 이유와 동시 실행 부하를 측정 구간에 맞춰 기록해야 원인을 좁힐 수 있다.

수정 범위: 기존 PC 환경은 유지하고, 다른 GPU 작업을 배제한 측정 창에서 baseline/candidate를 교대로 비교한다. 최초 초기화/워밍업은 별도 기록하며, 안정화를 확인한 측정값을 사용한다. 불리한 값만 사후 삭제하지 않는다. 같은 설정의 반복 편차가 후보 효과와 비슷하면 tie/inconclusive로 남긴다. 드라이버 업그레이드·임의 clock 고정·새 solver는 필요하지 않다.

## 5. FP32 결과: 성능 동기는 강해졌지만, 정확도 보정 후보의 결과는 아니다

### 5.1 같은 선택 launch profile에서의 측정

RTX 5070의 `fp64_hilo_selected`와 `fp32_hilo_selected`는 동일한 (32,32,256) profile을 사용한다. 각 run의 3회 중앙값으로 계산했다.

| 역할 | FP64 hi/lo (µs/call) | FP32 hi/lo legacy (µs/call) | 측정 비율 |
|---|---:|---:|---:|
| volume | 693.611 | 45.354 | 15.2934× |
| interior edge | 501.354 | 38.363 | 13.0686× |
| boundary edge | 252.389 | 30.771 | 8.2021× |
| assembled_force | 1830.051 | 416.992 | 4.38870× |

이것은 FP32 활용을 조사할 실질적인 동기다. 하지만 최종 solver 속도, 이론 FP32/FP64 연산기 비율, 정확도 보정 경로의 성능과는 다르다.

단일 `fp32` 변형은 scalar dtype 외에 `two_sum`, `two_product`, `pair_add`, `pair_scale`을 단순 연산으로 바꾼다. 근거: `R/evaluation/teacher_precision_compare.py:87–100`. 따라서 단일 FP32 대 FP64 hi/lo의 비율은 정밀도와 보상 연산량이 함께 바뀐 결과다.

### 5.2 기존의 안정화 경로가 빠져 있다

근거:

- `R/evaluation/teacher_dual_gpu.py:241–253`: `specialize(..., diagnostic=False)` 호출.
- `R/evaluation/teacher_precision_compare.py:50–61`: 기본 `strain_formula='legacy'`; stable, stable_normal_pair, stable_geometry_pair, stable_metric_pair 분기가 이미 존재.
- `W/precision_map.json`: FP32 hi/lo를 legacy로 명시.
- 동봉된 `fp32_hilo/code/.../p3_shell_warp_precision_kernels.py`: pair 기하를 합친 뒤 legacy constitutive 경로 사용.

이는 새 알고리즘을 구현해야 하는 상황이 아니라, **이미 있는 보정 경로를 fixed-work harness에 연결해야 하는 상황**이다. 어떤 stable 변형이 프로젝트의 현재 canonical FP32 후보인지 먼저 기존 실험 설정에서 확인하고, 정확한 이름과 소스 지문을 별도 variant로 기록한다. legacy 실측을 삭제하거나 보정 경로 결과로 이름만 바꾸면 안 된다.

RTX 5070 FP32 hi/lo legacy의 초기 상태 힘 차이:

\[
e_{f,\rm rms}=3.883372\times10^{-6}\;\mathrm N,\quad
e_{f,\infty}=2.270353\times10^{-5}\;\mathrm N,
\]
\[
e_{f,\rm rel2}=1.409757\times10^{-2}\approx1.410\%.
\]

상대오차 분모는 FP64 free-node 힘 배열의 L2 norm이고, RMS는 free-node 전 성분 RMS다. 이 값은 **동일 입력의 힘 평가 차이**다. 시간 적분 종료 시의 **동적 평형 잔차**와 같은 양이 아니다. 이전의 2.531e-9 N 잔차와 직접 비교하여 구현 퇴행을 선언해서는 안 된다. 반대로 유한한 출력이라는 사실도 현재의 엄격한 teacher 검산 적합성을 의미하지 않는다.

### 5.3 Eager CUDA-event 구간과 순수 kernel duration을 구분해야 한다

근거: `R/evaluation/teacher_dual_gpu_worker.py:143–180`.

```python
wp.record_event(a)
for _ in range(calls):
    wp.launch(kernel, ...)
wp.record_event(b)
ms = wp.get_event_elapsed_time(a, b)
```

`assembled_force`도 Python loop에서 `ops.evaluate()`를 호출하며, 그 안에 다수의 launch와 진단 연산이 있다. CUDA event는 stream에서 두 event가 실행된 시각의 차이다. Python이 다음 작업을 제출하기 전에 GPU가 기다리면 그 빈 시간도 구간에 포함될 수 있다. 따라서 “GPU clock으로 측정”은 “CPU 제출 간격이 제거된 순수 kernel duration”과 동일하지 않다. 특히 수십 µs 수준 FP32 및 여러 작은 launch의 묶음에 중요하다. [W1, W3]

현재 수치를 무효 처리할 필요는 없지만 `eager_event_batch`로 명시하고, 원래 생산 경로와 더 가까운 **고정 호출 수의 pre-captured CUDA Graph replay event 측정**을 추가한다. capture/compile/warmup 비용과 per-call 로그 추가는 측정 밖에 둔다. Graph가 각 반복의 데이터 의존성과 출력 쓰기를 보존하는지 실제 node 수와 출력으로 확인한다. 원래 측정값은 유지한다.

또한 `teacher_dual_gpu.py:303–316`은 같은 precision/role/block의 모든 행을 합쳐 비율을 만들므로 `_selected` run이 동일 block 집단에 추가된다. 1080 Ti에서는 초기/후반 편차가 커 통합 중앙값이 block 순위를 바꾸기도 한다. report에는 run/group별 n, block profile 전체, eager/graph 모드를 표시하고, 중복·불균형 pooled sample로 얻은 `measured_best`를 고정된 최적값처럼 표현하지 않는다.

## 6. 실제 시험 구간: 비영 바람 구간이 아니다

근거:

- `config.json`: frames=1, start_frame=0, fps=60, substeps=64.
- `input/inputs/forcing.npz`: `wind[0]=[0,0,0]`, `gravity[0]=[0,0,-9.81]`.
- `R/evaluation/teacher_dual_gpu_worker.py:251–257`: frame i에서 `forcing[0][i]`, `forcing[1][i]`를 그대로 사용.
- `input/checkpoint.npz`: 비영 u 및 v 존재. 최대 |u_hi| 약 1.3177e-4 m, 최대 |v_hi| 약 1.1626e-4 m/s.

따라서 이 결과를 “바람이 켜진 한 프레임 검증”으로 서술하지 않는다. 기존 속도가 있으므로 공기에 대한 상대 운동까지 0이라고 단정할 수도 없다. 정확한 표현은 **preload 이후, 부과된 바람은 0인 중력/기존 운동의 한 프레임**이다.

측정값은 유효하지만 nonlinear 난이도·외력 규모·대변형·장기 drift를 포괄하지 않는다. 다음 검증에서는 기존 궤적의 비영 바람·충분한 변형이 있는 저장 상태를 사용하고, 그 상태의 정확한 외력 시각을 함께 복원한다. 미래 바람 벡터만 초기 checkpoint에 끼워 넣고 동일 궤적이라고 부르면 안 된다. start_frame 설정이 실제 indexing과 시간 기록에 반영되는지도 확인한다.

새 모델이나 장시간 정밀도 개발보다, 현재 5070 FP64 후보의 대표 구간 재검증이 우선이다.

## 7. 두 PC의 수치 호환성: 작은 차이지만 입력 계수 지문 주의

ZIP에는 collect 결과가 없어 검토자가 저장 NPZ로 비교했다. 아래는 각 PC `baseline_r0`, 끝 시각 1/60 s의 free-node 성분 L∞이다.

| 양 | GTX 1080 Ti vs RTX 5070 |
|---|---:|
| 변위 (m) | 1.382748e-17 |
| 속도 (m/s) | 4.122249e-13 |
| 힘 (N) | 3.341606e-13 |
| 탄성 에너지 (J) | 1.998998e-19 |
| 운동 에너지 (J) | 6.409357e-22 |

두 PC candidate끼리 비교해도 같은 규모다. baseline 안의 반복 차이와 비슷한 수준이며, 이 짧은 구간에서 큰 장치 간 분기는 관찰되지 않는다. **장기 label 병합 적합성은 미판정**이다.

원본 checkpoint·forcing·plan·동봉 수치 소스는 동일하다. 그런데 `W/environment.json`의 `quadrature_input_hash`는 GTX `888a20...`, RTX `5db6ee...`로 다르다. `teacher_dual_gpu_worker.py:286–288`에 따르면 이 값은 model.volume의 ids/G/H/weights 각 배열의 실제 바이트로 계산한 지문이다. 따라서 적어도 하나의 생성 배열이 byte-identical하지 않다. 차이의 크기나 원인은 이 hash만으로 알 수 없다. CPU/BLAS preprocessing의 last-bit 차이일 가능성도 있지만 미확인이다.

`teacher_dual_gpu.py:131–139`의 collect matching은 원본 case/source/backend만 확인하고 이 actual hash를 검사하지 않는다. 교차 결과를 버릴 필요는 없지만 `matched_original_inputs=true`, `matched_preprocessed_arrays=false`처럼 분리해야 한다. 필요하면 기존 배열을 내보내 ids는 exact 비교, G/H/weights는 절대·상대 차이를 보고한다. 환경 재구축으로 범위를 늘리지 않는다.

1080 Ti의 시간 변동과 CPU·OS·스레드 환경 차이 때문에 이번 자료의 PC 시간 비율을 GPU만의 속도 차이로 해석하지 않는다. 이전에 언급한 1.2배를 이 데이터로 확정하거나 반박하기 어렵다.

## 8. 캐시 구현과 산출물 상태

`R/teacher/force_launch_profile.py:22–56`에는 baseline/explicit/cached, identity 검증, checksum, atomic rename, fallback 구조가 있다. 검토자가 이 순수 CPU 함수에 대해 실행한 합성 테스트 11개는 모두 통과했다. cache hit fixture, missing, stale identity, checksum, schema, precision, invalid block, unvalidated, malformed JSON 등을 포함한다. 결과: `config_unit_tests.json`.

그러나 양쪽 `selected_launch_profile.json`은 실제 profile이 아니라 baseline_retained 사유를 담고 있다. 이를 `--profile`에 넣으면 유효 schema가 없어 baseline으로 돌아가는 것이 정상이다. **실제 후보를 채택한 profile을 읽고 GPU 실행까지 이어지는 cache hit 통합시험은 이번 ZIP에서 확인되지 않는다.** 순수 함수 단위시험과 분리해야 한다.

동일 반복 내 actual preprocessing hash가 달라지면 cache miss가 생길 수 있으므로, profile hit/miss 이유를 기록하고 동일 PC 반복에서도 preprocessing 지문이 안정적인지 확인한다.

루트의 `fixed_work.csv`/`fixed_work_status.json`은 이전 준비 산출물의 빈 상태가 남아 있다. 실제 측정값은 `workers/<gpu>/fixed_work.csv`에 존재한다. 결과 취합은 workers 경로를 canonical로 명시하여 “FP32 미측정”으로 오독되지 않게 한다.

## 9. 후속 작업 우선순위 — 새 solver 개발 불필요

| 우선 | 작업 | 완료 조건 | 변경 금지 |
|---|---|---|---|
| P0 | 회귀 판정과 보고 수정 | exact equality, 기존 audit, baseline 반복 편차, 승인된 회귀 예산을 분리. 5070의 성능 성공을 그대로 보존 | 새로운 완화 tolerance 임의 도입, 자동 training_eligible 승격 |
| P0 | RTX 5070 32/32/256 대표 비영 바람 구간 재검증 | 실제 checkpoint·외력 시각 일치, 기존 검산 및 반복량 비교, 짧은 연속 구간 시간 | 물리식·dt·수렴 기준 변경 |
| P1 | GTX 1080 Ti 측정 안정화 | 동일 workload에서 반복 편차 확인, baseline/candidate paired 결과, clock/온도/부하 로그 | 실패 시간 사후 삭제, 근거 없는 최적 block 확정 |
| P1 | fixed-work 측정 모드와 FP32 variant 보완 | eager vs pre-captured Graph 분리, 기존 canonical stable FP32를 별도 측정, 입력·출력 오차 확인 | 새 혼합 solver, legacy 결과 이름 바꾸기 |
| P2 | 실제 profile 및 collect 검증 | 승인된 로컬 profile의 cache hit/stale/invalid integration, 실제 전처리 배열 차이 표시 | 새 분산 프레임워크, 장치 간 compiled cache 공유 |

현재 권고: RTX 5070은 (32,32,256)을 **유망한 FP64 후보**로 유지한다. GTX 1080 Ti는 baseline을 유지하며 측정부터 보강한다. 두 GPU를 같은 block으로 강제하지 않는다. FP32 전환을 포기할 이유는 줄었지만, 빠른 legacy 결과를 현재 teacher 품질로 사용할 근거는 아직 없다.

## 10. Codex 전달용 추가 요청

```text
이번 dual_gpu v1 결과는 폐기하지 말고, 아래 누락과 판정 문제만 보완해.
새 solver, 물리식 변경, 허용오차 완화, FMA/fast_math 변경은 하지 마.

1) RTX 5070의 32/32/256 후보는 기존 검산을 통과하며 solver+audit
   1.274배 가속을 보였다. baseline_retained 이유를 성능 실패나
   기존 audit 실패로 표현하지 마.

2) teacher_dual_gpu.py의 linf==0 필수 채택 조건을 재검토해.
   exact equality는 관측 항목으로 분리하고 baseline 반복 편차를
   같이 보고해. 기존 회귀 예산이 있으면 적용하고, 없으면
   approval/decision 상태를 남기되 새 완화값을 임의로 만들지 마.
   training_eligible이나 생산 기본값은 자동 변경하지 마.

3) 이번 forcing[0]은 wind=[0,0,0]이다. 비영 바람과 충분한 변형의
   기존 checkpoint를 선택하고 정확한 외력 시간도 복원해,
   5070 baseline/32-32-256을 같은 구간에서 검산 포함 비교해.
   checkpoint와 맞지 않는 미래 바람만 붙이지 마.

4) 1080 Ti는 동일 설정에도 큰 시간 변동이 있다. 다른 GPU 작업이
   없는 측정 창에서 clock/temperature/power/utilization을 기록하고
   준비·워밍업을 분리한 paired baseline/candidate 측정을 먼저 해.
   차이가 반복 편차보다 작으면 inconclusive 및 baseline 유지.

5) 기존 eager fixed-work 결과를 보존하고 명시적으로 표시해.
   동일 C회 호출을 pre-captured Graph에 넣어 event로 재측정하여
   Python 제출 공백이 성능비에 미치는 영향을 분리해.
   capture/compile/reset/추가 logging 비용은 별도로 기록해.

6) FP32는 이번 legacy 경로 외에 현재 프로젝트의 canonical한
   기존 stable strain/geometry/metric 보정 경로를 별도 variant로
   연결해. 새 알고리즘을 구현하지 말고 실제 공식/연산 정밀도와
   source hash를 남겨. 전체 FP32 적분 완주는 요구하지 않아.

7) 교차 PC 비교는 원본 hash와 실제 ids/G/H/weights hash를 구분해.
   양쪽 actual quadrature_input_hash가 다른 사실을 숨기지 말고,
   배열 차이 확인이 가능하면 정량화해. cache hit/stale/invalid의
   실제 통합 실행도 가능한 범위에서 기록해.

핵심 산출물은 corrected_report.md, 원시 CSV/NPZ, 재현 명령,
정확한 variant/측정 모드 구분, 변경 diff다.
원래 v1과 새 결과를 혼합하지 말고 별도 v2 폴더/ZIP으로 남겨.
```

## 11. 외부 기술 근거

아래는 CUDA 측정·수치 해석에 대한 일반 근거다. 이 프로젝트의 시간·오차 값은 위에서 명시한 업로드 파일에서 재계산했다.

- [W1] NVIDIA, CUDA C++ Best Practices Guide, §7.1.1 reference comparison 및 §9.1 timing. CUDA event는 stream에서 기록된 시각 차이이며, 참조 비교의 기준은 알고리즘에 맞게 정의한다. `https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html`
- [W2] NVIDIA, Nsight Compute Profiling Guide, clock control / measurement / reproducibility. Clock state, thermal throttling, concurrent GPU activity 등의 영향을 분리해야 한다. `https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html`
- [W3] NVIDIA Warp 공식 graph capture 예제. per-kernel launch overhead를 줄이기 위한 ScopedCapture/graph replay 사용을 설명한다. `https://github.com/NVIDIA/warp/blob/main/warp/examples/core/example_graph_capture.py`
- [W4] NVIDIA, Floating Point and IEEE 754. 연산 순서와 반올림을 구분해야 하며, 작은 결과 차이의 구체적 원인은 프로젝트 측정으로 확인해야 한다. `https://docs.nvidia.com/cuda/floating-point/index.html`

## 12. 재계산 부속 파일

- `run_stats.csv`: GPU/variant별 시간 중앙값, MAD, min/max.
- `paired_timing.csv`: candidate vs 바로 뒤 baseline의 쌍별 비교.
- `fixed_stats.csv`: run별 fixed-work 시간; 원본 pooled ratio와 분리.
- `state_comparisons.csv`: baseline 상호·candidate 상호·교차 및 PC 사이 NPZ 차이.
- `audit_stats.csv`: 각 run의 기존 audit 필드 요약.
- `config_unit_tests.json`: reviewer CPU-only 합성 설정 시험; GPU 검증 아님.
- `recompute_results.py`: numpy/pandas 기반 오프라인 재계산 스크립트.

실행 예: ZIP을 각각 `<root>/gtx1080ti/`, `<root>/rtx5070/` 아래에 풀고,

```bash
python recompute_results.py --root <root> --out <derived_output>
```

원본 GPU 코드는 실행하지 않으며 CUDA/드라이버 설치가 필요하지 않다. 합성 cache 단위시험은 별도로 수행한 검토자 시험이므로 이 재계산 스크립트에서 자동 재실행하지 않는다.

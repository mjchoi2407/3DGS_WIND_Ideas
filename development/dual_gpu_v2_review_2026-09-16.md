# Dual-GPU teacher v2 독립 검토 및 다음 작업 제안

- 작성일: 2026-09-16.
- 입력: `dual_gpu_gtx1080ti_v2.zip`, `dual_gpu_rtx5070_v2.zip`.
- 검토 범위: 두 ZIP의 CSV·JSON·NPZ 재계산, 실행용 동결 소스 대조, manifest 검증, 순수 CPU 판정 함수 검사.
- GPU solver/프로파일러를 이 검토 환경에서 재실행한 결과는 아니다.
- `corrected_report.md`의 요약을 그대로 수용하지 않고 원시 실행·배열을 다시 계산했다.
- 각 ZIP 내부 기준 경로를 아래에서 사용한다. 두 ZIP의 파일 경로 구조는 같다.

## 1. 결론

**5070의 FP64 블록 튜닝은 실제 바람에서도 유효하지만, 개선폭은 C0의 21.57%에서 W1의 6.89%로 감소한다. 1080 Ti의 64/64/64 후보에는 채택 근거가 없다. 보정된 FP32 힘 평가가 5070에서 약 9.8배 빠르다는 사실은 확인됐지만, 전체 Newton/GMRES를 포함한 FP32 teacher 가속은 아직 측정되지 않았다.**

다음 개발 우선순위는 추가 force block sweep/범용 cache 확장이 아니라 **W1의 선형 풀이 비용 분해와 실제 선형화 상태에서 HVP/전처리 적용의 정밀도·속도 비교**다. 전체 FP32 전환, 허용오차 완화, 새 전처리기 도입은 해당 측정 이후에 결정한다.

## 2. 원시 자료의 완전성 및 검토 기준

- 5070 manifest 1,556개 항목, 1080 Ti manifest 1,596개 항목의 해시를 모두 재검증했다. 누락·불일치 0개.
- 두 PC의 C0/W1 각각 `input_source_hashes.json` 전체가 일치한다. 원본 입력·runtime·정밀도 variant 소스의 동일성이 확인된다.
- v1의 `runtime/code/wind3dgs/teacher/*.py` 88개를 v2 C0 기준 소스와 비교했다. 변경 파일은 `force_launch_profile.py` 하나이고 나머지 87개는 바이트 단위 동일하다. 이 범위를 넘어 저장소 전체 불변성을 주장하지 않는다.
- `workers/<gpu>/run_summary.csv`의 일반 실행 42회 모두 `passed=true`.
- 이 실행들의 audit NPZ 90개, 내부 스텝 5,760개에서 flags가 모두 0이고 checks는 유한하다. 반복 실행의 합계이므로 5,760개의 연속·독립 상태를 검증했다는 뜻은 아니다.
- 각 fixed-work CSV 480행 전부 finite이고 force_status=0이다.

## 3. FP64 실제 구간 성능

A=256/256/256. B=5070에서32/32/256, 1080 Ti에서64/64/64. 순서는 volume/interior_edge/boundary_edge.

| gpu       | case   |   runs_per_label |   baseline_median_s |   candidate_median_s |   speedup |   time_reduction_pct | all_passed   |
|:----------|:-------|-----------------:|--------------------:|---------------------:|----------:|---------------------:|:-------------|
| rtx5070   | C0     |                3 |             1.56289 |              1.2258  |  1.275    |             21.5685  | True         |
| rtx5070   | W1     |                6 |            20.4143  |             19.0079  |  1.07399  |              6.88908 | True         |
| gtx1080ti | C0     |                6 |             2.01614 |              2.08753 |  0.9658   |             -3.54113 | True         |
| gtx1080ti | W1     |                6 |            31.162   |             32.4109  |  0.961467 |             -4.00778 | True         |

시간 단위는 초이며 solver+independent audit를 포함한다. process 전체, 초기화, 컴파일, 별도 snapshot 진단, 저장을 모두 포함한 생산 wall-clock은 아니다. n은 A/B 각각의 반복 수다.

계산식:

\[
\widetilde T_A=\operatorname{median}_r T_{A,r},\quad
\widetilde T_B=\operatorname{median}_r T_{B,r},\quad
S=\widetilde T_A/\widetilde T_B,\quad
D=100(1-\widetilde T_B/\widetilde T_A).
\]

### 3.1 5070

W1 A의 전체 범위는20.383822~20.443793초, B는18.962706~19.017809초다. 여섯 쌍 모두 B가 빠르며 A/B는1.07305~1.07592다. 따라서 이 시험에서 관측된6.89% 감소는 측정 편차보다 충분히 크다. 별도의 장기·다중 workload 유의확률을 산출한 것은 아니다.

W1 solver 중앙값은19.755781→18.552541초, audit는0.657137→0.453362초다. solver 약6.09%, audit 약31.01% 감소. 검산 자체의 최적화는 유지되지만, 검산이 W1 baseline 전체의 약3.22%뿐이어서 전체 이득은 제한된다. 구간별 중앙값의 합을 전체 중앙값과 동일시하지 않는다.

C0에서 확보했던21.57%를 모든 바람 구간에 적용할 수 없다. 이번 W1의 실제 개선은6.89%다.

### 3.2 1080 Ti

C0/W1의 후보 중앙값은 baseline보다 각각3.54%/4.01% 더 길다. W1의6쌍 중5쌍에서 후보가 느리다. 정확한4% 회귀를 보편적으로 확정하기에는 분산이 있으나, **현재 후보를 성능 최적 설정으로 채택할 근거는 없다. baseline 유지가 적절하다.**

일반 실행 telemetry에서 온도는 C0 60~66°C, W1 63~68°C다. v1의83°C 표본을 이번 결과의 원인으로 재사용해서는 안 된다. W1 사용률90% 이상 표본의SM clock은1746~1911MHz다. 단, telemetry는 프로세스 구간 전체를 포함하며 solver 시작·종료의 정확한 timestamp가 없으므로 이 기록만으로 변동 원인을 특정할 수 없다. WSL 환경의 프로세스 조회가 모든 graphics 부하를 포착했다고 가정하지 않는다.

이번 단계에서1080 Ti의 새 block sweep을 반복할 필요는 없다. 실험 설정은 GPU별로 다를 수 있다. 5070은 B, 1080 Ti는 A를 유지하되 생산 default와 label 적격성 승격은 별도다.

## 4. W1은 실제 비영 바람이고 선형 반복량이 크게 증가했다

`cases/W1/config.json`, `input/checkpoint_provenance.json`, 각 `pair*/result.json`을 대조했다.

- forcing index60/61/62, 물리 시각3.000000~3.050000초의3프레임.
- wind=[0.574505,1.120464,0.437511], [0.422209,1.048579,0.447192], [0.269913,0.976694,0.456873]. 모두0이 아님.
- preload2초 + 원본 wind phase1초가 반영된 저장 상태라는 provenance와 worker indexing이 일치한다.
- C0와W1의plan에 담긴 material, bending_ratio=0.002, fps60, substeps64는 같다.
- 준비 코드는 원본 wind chunk의index60, phase1.0초를 확인하고 next forcing index60을 사용한다. 원본 chunk 전체는 ZIP에 없으므로 그 이전60프레임을 여기서 재생/독립 재검산한 것은 아니다.

| 항목 | C0 | W1 |
|---|---:|---:|
| 실행 프레임 | 1 | 3 |
| 내부 스텝 | 64 | 192 |
| 누적 GMRES 반복 | 192 | 10,630 |
| 스텝당 GMRES | 3 | 55.364583 |
| retry | 0 | 0 |
| 누적 분해/rebuild 카운터c[15] | 2 | 13 |

W1의 프레임별GMRES는3,283/3,420/3,927이다. 위 카운트는 양쪽GPU 및A/B의모든 일반 실행에서 같다. 카운터 의미는 `resident_step_kernels.py:75–76,88–90` 및 실제 병렬 경로 `resident_parallel_reductions.py:42–45`에서 확인했다. `c[9]`는선형 반복누계이며 `c[15]`는built횟수다. `c[0]`처럼마지막 상태의반복값을총Newton횟수로오독하지않았다.

\[
\frac{(10630/192)}{(192/64)}=18.454861.
\]

즉 바람 구간에서는 스텝당 선형 반복이약18.45배다. 이것은 선형 관련 비용을 다음 진단 대상으로 삼을 강한 이유다. **그러나 반복 수만으로 HVP·cuDSS 삼각 풀이·직교화의 실제 시간 비중을 확정할 수 없다.** v1 C0의 49.94% force 비중을 W1에 재사용하지 말 것.

## 5. 보정 FP32: 실제 성능과 오차

선택 variant는 `stable_metric_pair`다. `precision_variants.json`의 `unapproved_existing_candidate`는 이전 연구자료에서 미승인인 기존 후보라는 의미이며 적절하다. GPU 함수를 새로 발명한 것이 아니다.

근거:
- `teacher_dual_gpu_followup.py:20–22,65–71`:variant연결.
- `corrected_precision_evidence.md`:기존 보정내역과과거엄격기준미통과.
- `teacher_dual_gpu_worker.py:127–147,192–230`:20회호출을한Graph내캡처하고replay를event로측정.

### 5.1 5070 / W1 / B, Graph replay, 호출당 μs

| role            |   fp32_hilo_corrected |   fp32_hilo_legacy |   fp64_hilo |   corrected_speedup |
|:----------------|----------------------:|-------------------:|------------:|--------------------:|
| assembled_force |              181.248  |           172.362  |    1776.62  |             9.80214 |
| boundary_edge   |               17.3472 |            15.2848 |     248.638 |            14.3331  |
| interior_edge   |               35.4144 |            31.8448 |     495.683 |            13.9967  |
| volume          |               24.8032 |            20.9376 |     687.638 |            27.7238  |

전체힘평가의보정 FP32가속은1,776.619/181.248=9.802배다. legacy172.362μs보다보정후181.248μs가약5.16%비싸지만가속대부분을유지한다. C0/B에서도1,790.298/183.794=9.741배다.

`assembled_force` Graph는20회정상호출에360kernel/40device copy/60memset을포함하고host copy·callback이없다. 이항목은volume/edge/조립등을포함한합성연산이므로하위커널시간과더하면중복이다. 모든fixed-work출력은A/B간및eager/graph간같은정밀도에서완전히일치했다. 저장된reset_bitwise_equal도true다.

반면1080 Ti 보정 FP32 assembled_force의 동일 profile 비교 가속은이번측정에서대략2.0~2.3배다. 예:W1/A는2,462.566/1,078.067=2.284배, W1/B는2,540.131/1,248.768=2.034배. 5070의9.8배를1080 Ti에전용하지말것. 순차측정·클록변동때문에세밀한정밀도별block순위도확정하지않는다.

### 5.2 동일 상태의 자유 DOF 힘 오차

| case   | variant             |         rms |        linf |          l2 |        rel2 |
|:-------|:--------------------|------------:|------------:|------------:|------------:|
| C0     | fp32_hilo_legacy    | 3.88337e-06 | 2.27035e-05 | 0.0002825   | 0.0140976   |
| C0     | fp32_hilo_corrected | 1.36072e-09 | 1.28375e-08 | 9.89871e-08 | 4.93974e-06 |
| W1     | fp32_hilo_legacy    | 8.32934e-06 | 4.98682e-05 | 0.000605927 | 0.000562587 |
| W1     | fp32_hilo_corrected | 1.33994e-08 | 1.74794e-07 | 9.74752e-07 | 9.05031e-07 |

\[
d_j=f^{32}_j-f^{64}_j,\qquad j\in\mathcal F,\quad N=|\mathcal F|,
\]
\[
e_\infty=\max_{j\in\mathcal F}|d_j|,\quad
e_{\mathrm{RMS}}=\sqrt{N^{-1}\sum_{j\in\mathcal F}d_j^2},\quad
e_{L_2}=\sqrt{\sum_{j\in\mathcal F}d_j^2},\quad
e_{\mathrm{rel},2}=e_{L_2}/\sqrt{\sum_{j\in\mathcal F}(f^{64}_j)^2}.
\]

여기서자유DOF는free노드의각좌표성분이며,N=5,292다. 위값은동일입력의내력평가차이다. time-step동적잔차,Newton종료오차,전체궤적오차가아니다. C0/W1 보정의상대L2오차는4.94e-6/9.05e-7,legacy대비L2오차감소는약2,854배/622배다. W1에서상대오차가작아도절대최대오차는1.75e-7N으로C0보다크다. 분모와물리량을함께봐야한다.

이결과로FP32teacher가공식잔차기준에합격한다고할수없다. 상태·힘·HVP·선형해오차와고정밀보정의수렴은별도다. **속도상이득이있다는것은입증됐으며,그이득을전체수렴에서유지할수있는지가남은핵심이다.**

## 6. 수치회귀 및 두PC 교차비교

### 6.1 5070 W1:모든 저장 시각 및 반복 조합의 최대 차이

| comparison   | quantity   |         rms |        linf |
|:-------------|:-----------|------------:|------------:|
| AA           | elastic_j  | 3.46945e-17 | 3.46945e-17 |
| AA           | force      | 1.20713e-13 | 7.17305e-13 |
| AA           | u          | 1.68306e-17 | 2.47002e-16 |
| AA           | v          | 1.26507e-13 | 1.0536e-12  |
| AB           | elastic_j  | 3.46945e-17 | 3.46945e-17 |
| AB           | force      | 1.25462e-13 | 7.2311e-13  |
| AB           | u          | 1.77674e-17 | 4.0087e-16  |
| AB           | v          | 1.30335e-13 | 1.26288e-12 |

AA는baseline모든쌍,AB는baseline6회×후보6회다. 단순pair00비교만이아니다. hi/lo는상태를먼저collapse하지않고

\[
\Delta u=(u^A_{hi}-u^B_{hi})+(u^A_{lo}-u^B_{lo})
\]

를extendedprecision에서계산했다. 속도도동일하다. 후보차이는baseline반복차와같은미세한규모이며,이구간에서의미있는수치열화증거는없다. 이것은장기오차보증이나새합격예산설정이아니다.

### 6.2 교차 PC: pair00의 A-A/B-B, 모든 저장 시각 최대

| profile   | quantity   |         rms |        linf |
|:----------|:-----------|------------:|------------:|
| A         | elastic_j  | 2.08167e-17 | 2.08167e-17 |
| A         | force      | 1.33236e-13 | 6.93421e-13 |
| A         | u          | 8.20139e-17 | 7.6411e-16  |
| A         | v          | 1.91214e-13 | 5.65721e-12 |
| B         | elastic_j  | 2.42861e-17 | 2.42861e-17 |
| B         | force      | 1.334e-13   | 6.2754e-13  |
| B         | u          | 7.78009e-17 | 8.11277e-16 |
| B         | v          | 1.9414e-13  | 5.69443e-12 |

W1의최대변위차는약8.12e-16m이하,속도차는약5.70e-12m/s이하다. 소스·원본입력은일치한다. 실제전처리arrays에서ids와weights는같지만G/H에차이가있으며,최대상대L2차이는1.83e-16이다. H의최대절대차이5.46e-12를단위·스케일없이큰오류라고부르면안된다.

두PC실행은매우가깝지만교차예산은여전히미정이다. 이검토에서재계산한cross값은bundle의`cross_device.csv`, `preprocessed_arrays.csv`에있다. worker보고서에교차검증미실행으로표시된것을GPU재실행성공으로바꾸지않는다.

## 7. 지시사항 반영 판정과 남은 소규모 보완

**주요 요청은 반영됐다.** 정확 일치를 필수 채택 조건에서 분리했고, 성능·기존 검산·수치 회귀·Graph·생산 상태를 별도로 보고한다. `save_profile`은 승인 예산 출처와 통과 근거가 없으면 저장을 거부한다. 비영 바람 W1의 3프레임, forcing offset, 균형 잡힌 실행 순서, telemetry, 보정 FP32, eager/Graph 측정 구분, 전처리 배열 내보내기, 준비용 빈 CSV 분리도 확인했다.

다음 항목들은 현재 성능 결론을 뒤집지 않는 후속 보완이다.

**판정 함수의 유한값 검사.** `teacher_launch_judgement.py:4`는 `status=='finite'`와 `linf is not None`만 검사한다. `linf=NaN, status='finite'`인 합성 입력에서 `comparison_status=complete`가 됐다. 실제 결과는 모두 유한하므로 이번 실행의 오류는 아니다. `math.isfinite`와 필수 필드·예상 비교 수 검사를 추가하면 된다. `judgement_smoke.json`에 재현 결과가 있다.

**성능 판정의 의미.** 같은 파일의 6–8행에서 사용하는 median±MAD 분리 조건은 선택용 휴리스틱이지 통계적 95% 보증이 아니다. 5070은 여섯 쌍 모두 명확히 빨라 현재 결론이 유지된다. 1080 Ti의 느린 후보도 `tie_or_inconclusive`라고만 표시되므로, signed 시간 차이와 쌍별 결과를 함께 보고해야 한다.

**상태 비교의 메타데이터.** `teacher_dual_gpu.py:77–83`에서 u/v 차분 자체는 올바르지만, `difference(zeros,d)`를 사용하므로 `reference_l2=0, zero_reference=true`가 항상 기록된다. 상대값은 의도적으로 비활성화했기에 절대오차는 문제없다. 상태 자체가 0이라는 뜻으로 오해하지 않도록 reference 메타데이터를 미정으로 두거나 원래 상태 norm을 별도 계산하면 된다.

**생산 보류와 성능 실험을 분리.** 승인 예산이 없어 production이 held인 것은 정상이다. 현재 `teacher_launch_judgement.py:11–13`은 승인 예산을 읽는 일반 기능 없이 미정/보류를 명시한다. 그렇다고 추가 성능 실험까지 막을 필요는 없다. 캐시·예산 API를 더 확장하는 일을 다음 최우선 작업으로 만들지 않는다.

**중요한 미측정 항목.** W1의 실제 커널 시간 분해와 HVP/전처리 fixed-work가 아직 없다. 이것이 다음 개발 결정을 위해 가장 필요한 자료다.

## 8. 다음 Codex 작업: W1의 선형 풀이에 집중

### P0. 최소 기준 고정

5070은 `32/32/256`을 다음 성능 실험의 reference candidate로 사용하고 원래 `256/256/256`도 보존한다. 1080 Ti는 `256/256/256`을 유지한다. `64/64/64`의 채택이나 전체 block sweep 반복은 중단한다.

물리식, 물성, 구적, dt, 허용오차, 전처리 알고리즘, FP64 검산 기준은 유지한다. 개발 중 실험 설정과 장기 teacher label의 생산 승격을 분리한다.

### P1. 5070/W1 실행 비용 분해

C0가 아니라 동일한 W1 3프레임에서 일반 wall-clock 측정과 진단 프로파일을 구분해 수집한다. HVP, cuDSS forward/backward solve, 직교화·내적, 참 잔차·line search, 힘 평가, 독립 검산의 비용을 구분한다. 상위 1~3개 연산만 추가 fixed-work 대상으로 삼는다.

프로파일러 시간을 생산 가속률에 섞거나 중첩 시간을 합산하지 않는다. 각 Newton 선형계의 GMRES 횟수, 목표 eta, 참 잔차, factor age/rebuild도 가능하면 기록하되 판정 정책은 바꾸지 않는다. **스텝당 GMRES가 18.45배라는 값은 Newton당 반복이나 조건수가 18.45배라는 뜻이 아니다.**

### P2. 대표 Newton 선형계 한 개를 고정해 비교

W1에서 실제 사용한 상태, 방향 벡터, RHS, 자유 DOF, 질량·dt 항, factor 식별자를 저장한다. 기존 FP64와 보정 FP32 경로가 이미 있는 연산에 한해서 같은 입력으로 비교한다. 없는 새 solver를 만들거나 최종 허용오차를 느슨하게 하지 않는다.

기준 선형계와 HVP 비교는 다음으로 정의한다.

\[
A_{64}\Delta q=b_{64},\qquad
 y_{64}=A_{64}v,\qquad
 y_{32}=A_{32}\operatorname{cast}_{32}(v),
\]
\[
e_{Hv}=\frac{\|\operatorname{cast}_{64}(y_{32})-y_{64}\|_2}{\|y_{64}\|_2}.
\]

기준 norm이 0이면 상대값은 미정으로 두고 절대차를 기록한다. 필요한 경우 동일 FP32 입력을 FP64로 평가하는 추가 대조로 입력 반올림과 연산 오차를 분리한다. HVP의 물리적 정의를 변경하지 않는다.

선형 보정의 참 잔차는 반드시

\[
r_{64}=b_{64}-A_{64}\Delta q
\]

로 평가한다. `cast64(A32)`의 잔차를 원래 A64의 참 잔차로 부르면 안 된다. 평가 목표는 같은 참 잔차 기준까지의 총시간이며, 추가 반복·변환·고정밀 보정 비용도 포함한다. 전체 FP32 궤적 완주를 이 측정의 선행조건으로 삼지 않는다.

### P3. 측정 결과에 따라 다음 하나만 선택

HVP가 주비용이고 기존 FP32가 유망하면 고정 선형 문제의 혼합 정밀도 보정을 검토한다. cuDSS apply가 주비용이면 apply와 재사용·보정 비용을 대상으로 삼는다. 직교화·작은 커널·동기화가 주비용이면 해당 실행 구조부터 다룬다. 이전 C0의 비중을 W1에 그대로 적용하지 않는다.

불필요하게 엄격한 선형 풀이가 의심되더라도 허용오차 변경은 별도 FP64 정확도 예산 실험에서 결정한다. 이번 진단 중 임의 완화하지 않는다.

SUNDIALS/KINSOL 공식 문서도 inexact Newton에서 선형 정확도와 비선형 수렴을 연결해 다룬다. 이는 해당 solver를 교체하라는 뜻이 아니라, 커널 하나의 속도만으로 전체 수렴 시간을 예측할 수 없다는 구분이다.

## 9. 원시 근거 위치

- 성능:`workers/<gpu>/run_summary.csv`, `paired_ratios.csv`, 각case/pair*/result.json.
- 반복:각result.json의rows[].attempts[].counts[9], counts[15].
- fixed-work:`fixed_work.csv` 및`<case>/fixed_<A/B>_<mode>_<variant>/result.json,outputs.npz`.
- 시간/상태:`snapshot_*.npz`, `audit_*.npz`, `step_stats.csv`.
- 환경:`<case>/environment/result.json,preprocessed_arrays.npz`, `logs/*_telemetry.jsonl`.
- 소스:`cases/<C0/W1>/runtime/code/wind3dgs/...`.
- 정확도variant:`cases/<case>/fp32_hilo_corrected/code`,`precision_variants.json`.

이보고서재계산은동봉`recompute_v2.py`로재현된다. 원ZIP을각각INPUT_ROOT/rtx5070과INPUT_ROOT/gtx1080ti로안전하게풀고다음을실행한다:

```bash
python recompute_v2.py --inputs INPUT_ROOT --out REVIEW_OUTPUT_NEW
```

보고서생성의입력자료는사용자가제공한ZIP이며,아래공식문서는일반적해석근거다.
- NVIDIA Nsight Compute Profiling Guide: clock control/profiling overhead/replay.
  https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html
- SUNDIALS KINSOL Mathematical Considerations §7.2.9: inexact Newton 선형정확도와수렴.
  https://sundials.readthedocs.io/en/latest/kinsol/Mathematics_link.html

## 10. 최종 판정

5070의 FP64 튜닝은 유효하고, 두 GPU에서 서로 다른 설정을 운영하는 것도 가능하다. 다만 1080 Ti를 억지로 같은 방식으로 개선하려는 작업은 중단하는 편이 낫다.

보정 FP32의 약 9.8배 힘 평가 가속은 추가 검토의 강한 근거다. 그러나 대표 바람 구간에서 크게 늘어난 선형 반복의 비용까지 다뤄야 teacher 전체 시간의 절감으로 이어진다. **다음 단계는 W1 선형 풀이의 비용 분해와 고정 선형 문제의 정밀도 검증이다.**

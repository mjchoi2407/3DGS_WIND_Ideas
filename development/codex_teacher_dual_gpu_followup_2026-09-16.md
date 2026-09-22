# Codex 후속 실행 지시서 — 두 GPU 결과 v2 보완

작성일: 2026-09-16  
대상: GTX 1080 Ti PC, RTX 5070 PC의 기존 P3 shell mesh teacher  
선행 자료: `dual_gpu_profile_review_2026-09-16.md`, `dual_gpu_gtx1080ti_v1.zip`, `dual_gpu_rtx5070_v1.zip`

## 0. 목표와 이번 작업의 경계

기존 v1을 폐기하거나 처음부터 새 최적화 체계를 만들지 않는다. **판정 로직과 측정의 누락을 보완하고, RTX 5070의 유망한 FP64 설정을 실제 비영 바람 구간에서 확인하며, 기존 보정 FP32 경로의 성능·오차를 측정**한다. 구현·실측·원시 산출물까지 수행한다. 계획이나 요약만 작성하고 끝내지 않는다.

v1 검토에서 확인된 출발점:

| 항목 | 확인된 결과 | v2에서 해야 할 일 |
|---|---|---|
| RTX 5070 FP64 | `(volume, interior_edge, boundary_edge)=(32,32,256)`, solver+audit 약 1.274배 가속, 약 21.52% 시간 감소, 기존 검산 통과 | 후보 유지, 대표 비영 바람 구간 추가 확인 |
| GTX 1080 Ti FP64 | `(64,64,64)` 후보이나 같은 설정의 시간 변동이 큼 | 최적 설정 확정이 아니라 측정 안정화부터 |
| 자동 채택 | 끝 상태 `linf == 0`이 필수 조건이고 baseline 자체도 반복 간 완전 일치하지 않음 | 정확 일치, 기존 검산, 회귀 판정, 성능, 생산 승격을 분리 |
| 시험 입력 | 현재 첫 프레임의 부과 바람은 `[0,0,0]` | zero-wind 대조군으로 보존; 실제 비영 바람 checkpoint와 시각을 일치시킨 사례 추가 |
| FP32 | fixed-work는 `strain_formula='legacy'`를 사용 | 기존 정확도 보정 경로를 별도 variant로 연결 |
| 측정 방식 | Python 반복 호출의 eager CUDA-event 측정 | 기존 값 보존 + 미리 캡처한 Graph replay 측정 추가 |

위 수치는 v1의 짧은 구간 결과다. 전체 데이터 생성, 장기 안정성, 다른 mesh의 결과로 일반화하지 않는다.

### 변경 금지

물리식, P3 요소, 구적 규칙, 물성, 경계 penalty, dt/재시도 정책, Newton/GMRES 허용오차, line search, 전처리 알고리즘, 생산 정밀도, 독립 검산 기준을 바꾸지 않는다. FMA 교체, `fast_math`/`fuse_fp` 변경, register cap, 새 혼합 정밀도 solver, kernel fusion, batching, 새 분산 프레임워크, 드라이버·CUDA·Warp·cuDSS 업그레이드는 제외한다.

기존 사용자 변경과 v1 파일을 보존한다. 생산 기본값과 `training_eligible`을 자동 변경하지 않는다. FP32 전체 시간 적분 완주나 추가 학습 실험은 이번 완료 조건이 아니다.

---

## 1. P0 — 자동 채택 판정과 결과 보고 수정

### 1.1 확인할 기존 위치

아래 줄 번호는 v1 ZIP 기준이다. 현재 저장소에서는 함수·표현식을 함께 찾아 대조한다.

- `runtime/code/wind3dgs/evaluation/teacher_dual_gpu.py:263–284`
  - `linf == 0`과 `adopted = improved and audit_ok and exact and graph_ok`
  - cached 경로에서도 `exact`를 필수로 사용하므로 함께 확인한다.
  - 현재 3회로 고정된 완료 조건은 실제 설정한 반복 수와 일치하도록 수정한다.
- `runtime/code/wind3dgs/teacher/force_launch_profile.py:47–56`
  - `save_profile()`이 audit/regression 상태를 무조건 `passed`로 기록하는 부분. 판정 로직을 수정한 뒤 미승인 후보까지 통과로 기록되지 않게 한다.

### 1.2 결과 상태를 분리한다

다음은 권장 의미이며, 기존 schema에 대응되는 필드로 구현해도 된다.

```text
performance_status: improved | tie_or_inconclusive | slower | not_measured
physical_audit_status: passed | failed | incomplete | not_measured
exact_value_equality: true | false | not_compared
numerical_regression_status: passed | failed | budget_not_defined | incomplete
graph_configuration_status: verified | mismatch | not_measured
production_promotion_status: not_requested | decision_required | blocked
```

`linf == 0`은 비교한 값의 정확 일치 여부로만 보고한다. 바이트를 직접 비교하지 않았다면 이를 bitwise equality라고 부르지 않는다. audit 실패, 성능 미개선, 예산 미정, 정확 일치 실패를 하나의 `baseline_retained` 사유로 뭉치지 않는다.

기존 승인된 **상태/출력 회귀 예산**이 있으면 그 출처와 norm·단위·적용 범위를 기록하고 그대로 적용한다. Newton 힘 잔차 기준을 출력 힘 차이나 궤적 오차의 허용치로 임의 재사용하지 않는다.

예산이 없으면 `budget_not_defined`로 남긴다. 임의 `allclose(atol=..., rtol=...)`, `10×baseline_noise`, 장치별 완화 tolerance를 새 합격 기준으로 만들지 않는다. baseline 자체 변동은 진단 자료이지 물리 정확도 기준이 아니다.

**공식 회귀 예산이 없더라도 explicit 후보의 실험과 성능 보고는 계속한다. 생산 승격을 보류하는 것과 성능 실험을 중단하는 것은 다르다.** `regression_status='passed'`를 꾸며 캐시를 통과시키지 않는다.

### 1.3 baseline 재현 편차와 후보 차이를 함께 기록한다

같은 장치·같은 사례·같은 시각에서 baseline–baseline, baseline–candidate, 가능하면 candidate–candidate를 비교한다. 자유 DOF와 배열 대응이 동일한지 확인한다. 프레임 끝뿐 아니라 저장된 각 프레임 경계를 비교한다.

hi/lo 상태는 먼저 단일 FP64로 합치지 말고, 비교용 고정밀 경로에서 다음을 계산한다.

\[
d_{i,c}=(h^A_{i,c}-h^B_{i,c})+(\ell^A_{i,c}-\ell^B_{i,c}),
\]
\[
e_\infty=\max_{i\in F,c}|d_{i,c}|,\qquad
e_{\rm rms}=\sqrt{\frac{\sum_{i\in F}\sum_{c=1}^3d_{i,c}^2}{3|F|}}.
\]

`F`는 자유 노드 집합이고 `c`는 xyz 성분이다. 변위·속도는 각각 자신의 hi/lo 배열로 계산한다. 힘은 원래 저장된 힘 배열끼리 차분한다. 에너지는 같은 정의의 scalar 값끼리 절대 차이를 기록한다.

상대 L2 오차는 다음과 같다.

\[
e_{\rm rel2}=\frac{\sqrt{\sum d_{i,c}^2}}{\sqrt{\sum (y^{\rm ref}_{i,c})^2}}.
\]

참조 norm이 0이면 분자와 분모를 그대로 기록하고 상대오차를 `undefined_zero_reference`로 둔다. 임의 epsilon으로 숫자를 작게 만들지 않는다. 아주 작은 참조 norm일 때도 절대오차를 함께 보고한다. 사용한 비교 dtype과 실제 유효 정밀도를 기록한다. `longdouble`이 두 PC에서 FP64보다 정밀하다고 가정하지 않는다. 정밀한 hi/lo 차분이 불가능하면 FP64로 조용히 축소하지 말고 해당 비교만 미측정으로 남긴다.

NaN/Inf, 누락 snapshot, 불일치 시각, 구간 미완료는 성공으로 처리하지 않는다. 빈 비교 목록에 대한 `all([])`이 통과로 이어지지 않게 시험한다.

---

## 2. P0 — RTX 5070의 비영 바람 사례 검증

### 2.1 사례는 우선 두 개로 제한한다

- **C0:** v1의 동일 checkpoint와 동일 zero-imposed-wind 구간. 기존 결과와 연결하는 대조 사례다. v1의 성능·검산 성공을 재서술하거나 사후 삭제하지 않는다.
- **W1:** 기존 궤적의 비영 바람 및 실제 변형이 있는 checkpoint에서 시작하는 짧은 연속 구간. 최초 목표는 출력 3프레임이다. 출력 간격과 내부 dt 정책은 원래 설정을 유지한다. 3프레임 결과를 장기 안정성 검증이라고 부르지 않는다.

저장된 적절한 W1이 없으면 원래 FP64 baseline으로 기존 forcing history를 정확히 따라 해당 시각까지 진행하고 checkpoint를 만든다. 두 GPU 비교에 쓸 checkpoint는 가능한 한 이 동일한 파일을 복사한다. 기존 baseline조차 그 구간을 완료하지 못하면 원래 허용된 재시도만 사용하고 원인을 기록한다. 새 dt·tolerance를 만들지 않는다.

### 2.2 checkpoint와 외력 시간 일치가 선행 조건이다

v1 worker의 `load():99–110`, `frame():251–257`은 forcing을 읽어 로컬 `i`로 사용하며 snapshot 시간도 0부터 기록한다. `start_frame` 값을 설정 파일에 적는 것만으로 올바른 offset이 적용된다고 가정하지 않는다.

각 사례에 다음을 기록한다.

```text
case_id, checkpoint_hash, checkpoint_source_time_s,
checkpoint_source_frame, forcing_hash, forcing_time_origin,
forcing_start_index, physical_interval_start_s, physical_interval_end_s,
fps, requested_frames, actual_wind_and_gravity_per_frame,
state_restore_fields, restart_equivalence_status
```

전체 forcing history의 index `j0`가 checkpoint 직후 첫 구간에 해당한다면 로컬 프레임 `i`는 `j=j0+i`를 사용한다. 사전에 forcing을 잘랐다면 로컬 index는 0부터 쓰되 원래 `j0`를 metadata로 보존한다. **offset을 빠뜨리거나 두 번 적용하지 않는다.** 구간의 시작/끝 프레임 convention도 기존 solver 정의와 맞춘다.

절대 출력 시각은 동일 출력 간격의 경우 `t0+(i+1)/fps`로 기록한다. 외력이 원래 다른 시각 convention을 사용하면 그것을 보존한다. `t0`는 실제 checkpoint의 시각이다. 변위·속도·가속도·hi/lo 등 기존 restart에 필요한 상태를 확인하고, 없는 값을 임의로 0으로 채우지 않는다. 기존 재시작 방식이 가속도나 cache를 재생성한다면 그 방식을 두 설정에서 동일하게 사용하고 기록한다.

가능하면 checkpoint 생성 직후의 연속 실행과 재시작 baseline의 첫 프레임을 비교해 시간/상태 복원이 맞는지 확인한다. 이미 같은 조건의 검증 결과가 있으면 재사용한다.

### 2.3 비교할 launch 설정과 반복

```text
A = baseline: 256/256/256
B = candidate: 32/32/256
순서 = volume / interior_edge / boundary_edge
```

C0은 수정한 harness의 재현 확인용으로 최소 3쌍, W1은 아래 공통 측정 방식으로 6쌍을 비교한다. 불필요한 전체 block sweep은 반복하지 않는다. W1에서 후보가 느리면 그 사실을 보고하고 C0와 W1을 구분한다. 한 후보의 실패를 고치려고 수치식을 변경하지 않는다.

각 실행에서 solver/audit 양쪽의 실제 block/grid를 확인한다. 후보별 Graph는 해당 설정으로 다시 생성한다. 구성 metadata만 보지 말고 기존 Graph introspection이 가능한 범위에서 실제 launch를 확인한다.

기존 검산, Newton/GMRES·rebuild·fallback·retry, 완료 물리 시간, 상태·힘·에너지 차이를 남긴다. 기존 audit를 통과하지 못한 실행은 유효 teacher 가속률에서 제외하되 실패 시간과 원인은 따로 보존한다.

---

## 3. P1 — GTX 1080 Ti는 측정 안정화 후 판단

같은 설정에서도 v1 시간 편차가 컸으므로, 당장은 baseline과 기존 `(64,64,64)` 후보만 비교한다. 5070의 `(32,32,256)`을 강제하지 않는다. 두 PC의 설정이 달라야 한다고 가정하지도 않는다.

1. 기존 환경을 유지하고 자신의 이전 시험 작업이 종료됐는지 확인한다. 승인 없이 다른 사용자 프로세스를 종료하거나 clock/fan/power 설정을 바꾸지 않는다.
2. 초기화·JIT·Graph 생성·워밍업을 별도 기록한다. 시간 측정 중 GPU 상태를 timestamp와 함께 저빈도로 기록한다. 권장 표본 간격은 0.5–1초이며 더 높은 빈도로 계측 부하를 만들지 않는다.
3. 기록 항목: GPU/core/memory clock, 온도, 전력, utilization, performance state, 조회 가능한 제한 이유와 동시 GPU 프로세스. 지원하지 않는 항목은 `unsupported`로 남긴다. 온도 하나만으로 throttling을 확정하지 않는다.
4. 같은 설정의 예비 반복과 시간 추이를 확인한 뒤 A/B paired 실험을 한다. 안정적인 측정 창을 확보하지 못하면 `unstable_measurement`로 남긴다. 환경 수리를 무한 반복하지 않는다.
5. 우선 C0을 6쌍 비교한다. 안정적인 결과가 확보되면 같은 W1에서 확인한다. C0조차 계속 불안정하면 W1 성능 판정은 보류하되 가능한 수치 검산·FP32 출력 검사는 별도로 완료한다.

### 공통 시간 비교 계약

A/B 6쌍은 `AB, BA, AB, BA, AB, BA`처럼 순서를 균형 있게 배치한다. 각 실행은 같은 checkpoint로 복원한다. JIT cache 등 재사용할 준비 조건은 동등하게 하고, 상태·history·numerical factor의 잘못된 공유는 금지한다.

후보를 먼저 튜닝한 warm 상태와 baseline 최초 실행을 비교하지 않는다. 정해 둔 준비 실행은 별도 기록한다. 불리한 표본만 사후 삭제하지 않는다. 사전에 정의한 명확한 실패/계측 오류가 있으면 raw 전체와 제외 사유를 남기고, 전체 포함 통계도 제공한다.

`T`는 동일한 물리 구간의 solver+audit wall time이다. 구간 경계에서 필요한 완료를 확인하되 매 커널 뒤 강제 동기화를 추가하지 않는다. 초기화·전송·추가 snapshot 진단·저장·프로세스 전체 시간도 별도 기록한다. 중첩 범위를 더하지 않는다.

\[
\widetilde T_A=\operatorname{median}(T_{A,j}),\quad
\operatorname{MAD}_A=\operatorname{median}|T_{A,j}-\widetilde T_A|,
\]
\[
S=\widetilde T_A/\widetilde T_B,\qquad
r_T=100(1-\widetilde T_B/\widetilde T_A),
\]
\[
S_j=T_{A,j}/T_{B,j},\qquad
r_{T,j}=100(1-T_{B,j}/T_{A,j}).
\]

B의 MAD도 동일하게 계산한다. n, median, MAD, min/max, 쌍별 비율과 실행 순서를 모두 보고한다. A/B 각 run 하나가 독립 단위이며 내부 64스텝을 64개의 독립 표본으로 세지 않는다. 차이가 변동과 구분되지 않으면 `tie_or_inconclusive`로 두고 baseline을 유지한다. 작은 표본으로 임의 성공 확률이나 통계적 동등성을 주장하지 않는다.

---

## 4. P1 — 보정 FP32를 Graph fixed-work로 측정

### 4.1 경로를 정확하게 식별한다

`teacher_dual_gpu.py:241–253`의 `specialize()` 호출과 `teacher_precision_compare.py:50–61`의 `strain_formula='legacy'` 기본값을 확인한다.

기존 stable strain/normal/geometry/metric 실험 설정·보고서·소스에서 현재 정확도 보정 후보를 찾고, **정확한 flags와 소스 fingerprint**를 기록한다. `stable_metric_pair`라는 이름만 보고 최종 후보라고 단정하지 않는다. 예전 큰 잔차 개선을 만든 설정과 현재 연결할 설정이 같은지 확인한다.

최소 비교군:

```text
FP64 hi/lo baseline formula
FP32 hi/lo legacy                  # v1과 이어지는 대조군
FP32 hi/lo existing stable variant # 실제 기존 보정 경로
```

canonical 후보가 명확하지 않으면 기존 보고서에서 근거가 있는 보정 후보 최대 2개를 이름 그대로 비교하고 `canonical_variant_unresolved`를 남긴다. 단일 FP32는 기존 harness에서 즉시 가능한 경우 보조군으로만 유지한다. 새 보정 알고리즘·새 혼합 solver를 만들지 않는다.

실제 실행 variant를 CLI/config에서 명시하고 report까지 전달한다. variant별 별도 출력 디렉터리와 module/JIT 구분을 유지한다. legacy를 실행하고 이름만 stable로 바꾸거나, 이전 specialization의 module이 재사용되면 안 된다. 생산 source는 덮어쓰지 않는다. 기존 `diagnostic=False`와 독립 검산 수행 여부를 혼동하지 않는다.

### 4.2 eager 측정을 보존하고 graph 측정을 추가한다

기존 `teacher_dual_gpu_worker.py:143–180`의 결과는 `measurement_mode=eager_event_batch`로 보존한다. 추가 모드는 `graph_replay_event_batch`로 구분한다.

동일한 고정 입력의 연산 C회 호출을 **미리 한 Graph 안에 캡처**하고, 워밍업 후 event 사이에서 replay한다. 시작값 `C=20`, timed batch 5회를 사용한다. 단 한 호출 Graph를 Python에서 C회 반복 launch하는 것으로 대체하지 않는다. 시간이 너무 짧아 C를 늘려야 하면 비교군에 동일하게 적용하고 새 측정 그룹으로 기록한다.

capture·compile·변환·외부 reset·로그 비용은 측정 밖에서 각각 기록한다. 단, `assembled_force`의 정상적인 zero/copy/assembly/checks는 **반드시 포함**한다. 반복 사이 정확성에 필요한 reset은 제외해 일을 줄이지 말고 포함 여부·별도 비용을 명시한다.

각 커널이 덮어쓰기인지 누적인지, 여러 호출이 같은 상태를 의도대로 평가하는지 확인한다. Graph의 실제 호출/커널 수와 eager/graph 출력 동등성을 확인하고, last-output뿐 아니라 kernel status·NaN/Inf도 검사한다. stream을 바꾸어 node를 겹치게 하거나 algorithm을 변경하지 않는다. Graph capture가 지원되지 않는 경로는 eager 결과와 오류를 남긴다. 프로파일러 설치나 library 업그레이드로 우회하지 않는다.

C회 호출 Graph를 event 구간 안에서 한 번 replay했을 때:

\[
t_{\rm call}[\mu s]=1000\,T_{\rm event}[ms]/C.
\]

같은 이벤트 구간에서 동일 Graph를 R회 replay했다면 분모는 `C×R`이고 R도 기록한다. 다만 Python 제출 공백을 줄이는 주된 방법은 C회 작업을 Graph 내부에 넣는 것이다. 이 값은 Graph 작업 묶음의 amortized elapsed cost이지 각 kernel의 순수 duration 합이라고 이름 붙이지 않는다.

### 4.3 비교 범위를 제한한다

각 GPU에서 baseline 256/256/256과 현재 후보 설정만 우선 비교한다. 모든 precision×보정식×4³ block 조합을 sweep하지 않는다. 역할은 volume, interior edge, boundary edge, assembled_force 네 개다. `assembled_force`에 앞의 세 역할이 포함되므로 네 행을 더하지 않는다.

C0 및 W1 시작 상태를 각각 비교한다. FPS나 forcing을 사용하는 완주 실험이 아니라, 해당 checkpoint의 기하·힘 평가 fixed-work임을 명시한다. 가능하면 W1 끝 상태도 보조 표본으로 사용할 수 있으나 필수 범위를 늘리지 않는다.

고정 입력·동일 block·동일 measurement mode의 FP64 대 FP32 비교를 주 결과로 한다. precision별 측정 최선 설정 비교는 부차적으로 분리한다. run 순서·환경이 다른 결과를 `precision/role/block`만으로 pooling하지 않는다. GPU/case/variant/full profile/mode/session/run별 통계를 보존한다.

### 4.4 출력 오차를 가속률과 함께 기록한다

FP64 원본 상태에서 기존 변환 함수를 이용해 FP32/FP32-hi-lo를 준비한다. 상태 표현, 기하 누적, 힘, assembly, 진단의 실제 precision과 출력 의미를 `precision_map.json`에 기록한다.

힘/기하/에너지 진단의 절대 최대·RMS·상대 L2, 참조 norm, 자유 DOF mask, units, finite/status를 기록한다. 힘 오차는 1절의 같은 norm을 쓴다. 필드마다 단위가 다른 기하 진단을 한 norm에 합치지 않는다.

동일 상태의 힘 차이를 Newton 동적 잔차, 장기 궤적 오차, teacher 학습 적합성과 동일시하지 않는다. `force_error != Newton_residual`을 report에 명시한다. 과거 잔차 숫자는 동일 입력·동일 norm·동일 연산량일 때만 정량 비교한다.

기존 FP64 평가기로 변환된 상태의 재평가가 즉시 가능하면 아래 보조 분해를 추가할 수 있다. 이것을 위해 새 solver나 새 데이터 구조를 만들지는 않는다.

\[
e_{\rm state}=f_{64}(q_{32}^{repr})-f_{64}(q_{64}^{repr}),
\]
\[
e_{\rm eval}=f_{32}(q_{32}^{repr})-f_{64}(q_{32}^{repr}),\qquad
e_{\rm total}=e_{\rm state}+e_{\rm eval}.
\]

`q^{repr}`는 해당 hi/lo 또는 단일값이 나타내는 상태다. 상태 이외 기하 계수·물성까지 반올림되었다면 `e_eval`에는 그 영향도 포함되며, 이를 순수 산술 오차라고 부르지 않는다. 벡터 오차끼리의 합은 위와 같지만 norm의 합이 같다고 가정하지 않는다. 불가능하면 total만 보고하고 보조 분해는 `not_measured`로 남긴다.

---

## 5. P2 — 두 PC 비교와 캐시 검증의 빈 부분만 보완

### 5.1 원본 동일성과 전처리 결과 동일성을 구분

원본 checkpoint/forcing/source hash, 실제 ids/G/H/weights의 dtype·shape·hash를 따로 기록한다. v1의 actual quadrature hash 차이를 숨기거나 원본 hash로 대체하지 않는다.

가능하면 해당 배열을 저장하여 ids는 exact 비교, G/H/weights는 필드별 절대·상대 오차를 보고한다. ids 또는 shape가 다르면 확인 없이 같은 DOF끼리 비교하지 않는다. 바이트 차이가 곧 큰 물리 차이라는 판정도 하지 않는다. 전처리 차이를 없애려고 환경이나 알고리즘을 바꾸지 않는다.

교차 결과 상태는 다음을 구분한다.

```text
matched_original_inputs
matched_numerical_source
matched_preprocessed_arrays
same_physical_timestamps
cross_device_metrics
cross_device_acceptance_budget
```

각 PC의 baseline–candidate 비교가 먼저이며, PC 간 비교는 그 다음이다. approved 교차 예산이 없으면 `budget_not_defined`를 유지한다. 미완료·서로 다른 시각 결과를 비교하지 않는다. OS/CPU/환경 차이가 있는 PC 전체 시간 비율을 GPU 순수 성능비로 부르지 않는다.

### 5.2 캐시는 실험 증거를 위조하지 않는다

회귀 예산이 없는 후보는 evidence와 candidate blocks를 저장하되 approved profile로 위장하지 않는다. explicit 실험은 계속 가능해야 한다.

실제 승인된 로컬 profile이 있으면 cache hit 후 Graph가 올바른 block을 쓰는지, stale/source/precision/workload 불일치와 invalid 파일에서 baseline으로 돌아가는지 통합시험한다.

승인 profile이 없으면 정상적인 production-cache hit 검증은 `blocked_by_regression_budget`로 남긴다. 기존 unit-test fixture를 이용한 hit 분기 시험은 격리된 테스트 디렉터리에서만 수행하고 `fixture_only`로 명시한다. 합성 fixture 시험을 실제 후보 채택이나 teacher 정확도 통과로 보고하지 않는다. fallback 경로와 explicit 후보 실행은 실제로 검증할 수 있다.

새 분산 autotuner, GPU 사이 compiled cache 공유, mesh 크기 자동 일반화는 구현하지 않는다.

---

## 6. 원시 결과·보고서·재현 명령

v1과 분리된 v2 폴더/ZIP으로 저장한다. 실제 측정의 canonical 경로는 `workers/<gpu>/...`로 명시한다. 루트의 예전 빈 CSV가 최신 결과처럼 보이지 않도록 제거하거나 deprecated pointer로 교체한다. v1 원본은 변경하지 않는다.

권장 구성은 다음과 같다. 기존 도구와 호환되는 동등 이름도 가능하나 보고서에서 경로를 명시한다.

```text
dual_gpu_followup_v2/
  corrected_report.md
  experiment_manifest.json
  changes.diff
  repro_commands.md
  cases/{C0,W1}/case_manifest.json
  workers/<gpu>/
    environment.json
    run_summary.csv
    paired_timing.csv
    step_stats.csv
    telemetry.csv
    fixed_work.csv
    precision_map.json
    variant_manifest.json
    error_metrics.csv
    decision_status.json
    cache_tests.json
    snapshots/...
    logs/...
  comparison/
    input_match.json
    preprocessed_array_metrics.csv
    cross_device_metrics.csv
```

CSV 최소 식별자: worker/GPU, case_id, physical interval, run/session/pair ID, 실행 순서, precision variant, full block profile, eager/graph mode, calls, expected/completed frames, finite/audit 상태. 미측정은 0으로 채우지 않고 null/상태/이유로 남긴다.

v1 수치 재해석과 v2 실측은 별도 표로 제시한다. source/config/input/output hash와 실제 실행 명령을 남긴다. GPU별 실행·collect·재검증 명령이 있어야 한다. 계산한 통계는 raw CSV에서 재현 가능해야 한다.

### 최종 보고서 첫머리에서 답할 질문

1. 5070의 32/32/256 후보가 **비영 바람 W1**에서도 기존 검산을 유지하며 빨라졌는가? solver+audit 및 별도 setup/저장 시간은 얼마인가?
2. 1080 Ti의 측정 변동은 줄었는가? 64/64/64가 재현성 있게 이득인가, tie/inconclusive인가?
3. `linf == 0` 문제를 어떻게 분리했고, 수치 회귀·성능·생산 승격 중 무엇이 통과/미정인가?
4. **기존 보정 FP32**의 Graph fixed-work 속도와 동일 상태의 힘/기하 오차는 어떠한가? legacy 및 FP64와 무엇이 다른가?
5. checkpoint–forcing 시간, 실제 전처리 배열, cache-hit/fallback에 남은 미확인 사항은 무엇인가?
6. 다음 한 가지 우선 작업은 무엇인가? 실측으로 근거를 제한하고 전체 FP32 완주 성공률 등을 추정하지 않는다.

---

## 7. 실행 순서·중단 조건

우선순위는 **판정/시간-index 오류 보완 → 5070 W1 확인 → 1080 Ti 안정화와 기존 보정 FP32 측정 → collect/cache 빈 부분 검증**이다. 승인된 두 PC 실행 경로가 있으면 서로 독립된 작업은 동시에 수행해도 되지만, 한 GPU에서 benchmark를 서로 겹쳐 실행하지 않는다.

한 PC만 접근 가능하면 로컬 구현·실측을 완성하고, 다른 PC의 정확한 실행 명령과 collect 절차를 남긴다. 원격 접속 수단을 새로 만들거나 미실행 GPU 값을 추정하지 않는다. 회귀 예산 미정이나 NCU 미지원 때문에 나머지 실험을 중단하지 않는다.

같은 원인의 수정·재실행이 두 번 연속 실패하면 그 후보/측정만 중단하고 로그를 보존한다. 가능한 다른 결과는 완료한다. 안정적 측정 실패는 baseline 유지와 명시적인 미판정으로 끝낼 수 있다. 장시간 물리 복구, 환경 재구축, 새 solver 연구로 전환하지 않는다.

**이번 완료는 모든 후보의 성공이 아니라, 기존 FP64 성능 이득의 적용 범위와 보정 FP32의 실제 성능·오차를 재현 가능한 증거로 분리하는 것이다.**

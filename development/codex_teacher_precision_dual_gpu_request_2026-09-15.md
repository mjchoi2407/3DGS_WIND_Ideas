# Codex 실행 지시서 v2 — 두 PC 공통 mesh teacher 성능 진단 및 GPU별 튜닝

- 작성일: 2026-09-15 (KST)
- 대상: GTX 1080 Ti PC와 RTX 5070 PC
- 단계: Phase 1 / P0·P1 / FP64 launch 튜닝 + 기존 FP32 fixed-work 측정
- 대체 문서: `codex_teacher_precision_phase1_request_2026-09-15.md`
- 참고 문서: `teacher_precision_analysis_2026-09-15.md`
- 이 문서는 실행 요청이다. 새 성능 측정 결과나 완성된 구현이 아니다.

## 0. 수행 원칙과 완료 범위

현재 저장소에서 필요한 최소 변경을 구현하고, 접근 가능한 PC에서 실제로 실행하여 원시 결과를 남겨라. 분석 보고서를 다시 요약하거나 계획만 제시하고 끝내지 마라.

두 GPU에 공통으로 적용할 물리·수치 코드는 한 벌로 유지한다. GPU별로 달라도 되는 것은 이번에 허용한 launch configuration과 그 측정·선택 정보다. 두 GPU의 최적 블록 크기가 다르다고 가정할 필요도, 같게 맞출 필요도 없다. 각각 측정해 결정한다.

이 작업은 다음 네 질문에 답한다.

1. 각 PC에서 FP64 hi/lo와 기존 검산을 유지한 채 block_dim만 바꾸면 solver+audit 총시간이 줄어드는가?
2. 각 GPU에서 동일 입력·동일 작업량의 기존 FP32 경로는 얼마나 빠르며, 출력 오차는 얼마인가?
3. 측정 결과를 GPU·실행 환경·작업 크기별로 저장하고, 검증된 설정만 명시적 옵션으로 재사용할 수 있는가?
4. 두 PC에서 같은 기준 문제를 실행했을 때, 각자의 baseline 대비 정확도와 PC 사이 결과 차이는 어떠한가?

이 문서의 작업 범위가 이전 실행 지시서보다 우선한다. 단, 저장소의 공식 수치 계약·frozen-source 보호·사용자 변경은 이 문서로 해제되지 않는다. 분석 보고서의 옛 source line 대신 현재 함수와 호출부를 찾아 대응 관계를 기록하라. 충돌하는 부분만 중단하고 나머지 측정은 수행하라.

## 1. 이번에 변경하지 않을 것

다음은 양쪽 PC에서 공통으로 유지한다.

- Koiter/StVK shell 물리식, P3 요소, 구적 규칙, 재료·두께·penalty·경계 조건·외력.
- 적분기 선택, dt·재시도 정책, Newton/GMRES 허용오차, line search, 전처리 알고리즘.
- 생산 경로의 상태·힘·접선/HVP·선형 풀이·독립 검산 정밀도와 판정 기준.
- 기존 데이터의 `training_eligible` 등 자격 메타데이터. 성능 시험이 끝났다는 이유로 true로 바꾸지 않는다.

이번에는 전체 FP32 솔버 완주, 새로운 혼합 정밀도, TwoProduct-FMA 교체, fast_math/fuse_fp 변경, 강제 register cap, 새 전처리기, kernel fusion, 다중 궤적 batching을 구현하지 마라.

CUDA/Warp/cuDSS/드라이버/프로파일러의 업그레이드·다운그레이드, OS 전환, 전력·클록 설정 변경도 제외한다. 기존 환경에서 안 되는 부분은 호환성 문제로 분리하여 보고하고, 몰래 다른 backend나 CPU fallback으로 대체하지 마라.

다중 PC 분산 스케줄러, MPI/NCCL, 한 선형계의 GPU 간 분할은 범위 밖이다. 두 PC는 같은 프로그램을 각자 실행하는 독립 worker다.

기존 사용자 변경과 기준 결과를 보존한다. 생산 기본값은 자동 변경하지 않는다. 모든 후보는 실험 옵션으로 켜고 끌 수 있어야 한다.

## 2. 단계 A — 양쪽 실행 환경과 접근 가능성 확인

### 2.1 환경은 강제로 통일하지 않는다

각 PC에서 실제 GPU, CPU, OS/WSL 여부, 실행 backend와 라이브러리를 조회하라. GPU 이름을 보고 SM 수나 compute capability를 코드에 하드코딩하지 마라.

`environment.json`과 `compatibility.json`에 최소한 다음을 남겨라.

| 구분 | 기록 항목 |
|---|---|
| PC/GPU | worker_id, GPU 이름·UUID·compute capability·SM 수, GPU 메모리, CPU, OS/WSL |
| 소프트웨어 | Python, Warp 버전·설치 패키지, 실제 CUDA runtime/NVRTC 또는 컴파일러, cuDSS, 관련 cuBLAS, 드라이버 |
| 실물 경로 | 확인 가능한 실제 로드된 라이브러리 경로와 JIT target, 환경/패키지 잠금 정보 |
| 소스 | git commit, dirty 여부, 변경 diff, 공통 수치 코드·컴파일 옵션의 해시 |
| 실행 정책 | backend, CUDA Graph 사용 방식, stream, CPU thread 설정, 라이브러리 solver 옵션 |
| 지원 판정 | 장치 인식, 대상 커널 실행, cuDSS 사용 단계, 실제 baseline 실행, 프로파일러 지원 여부 |

`nvidia-smi`의 CUDA 표시만으로 설치된 toolkit/runtime 버전을 확정하지 마라. 시스템 nvcc와 Warp가 실제 사용하는 컴파일·런타임 경로도 구분하라.

CUDA 12.9는 공통 환경을 검토할 때의 후보이지 이번 작업의 설치 명령이 아니다. CUDA 13부터 Pascal 대상 offline compilation과 라이브러리 지원이 제거되었다는 점을 고려하되, CUDA 12.x라는 이름만으로 Warp/cuDSS/Graph까지 모두 작동한다고 단정하지 마라. [S1]

각 PC에서 현재 작동하는 경로를 우선 보존하고, 의존성 지원은 해당 설치 버전과 실제 실행으로 확인한다. import 성공이나 장치 인식만으로 전체 solver 지원을 통과시키지 마라.

두 PC의 라이브러리/컴파일 환경이 다르면 동일 소스 실행은 가능하더라도 GPU 자체만의 성능 비교는 아니므로 차이를 명시하라. 환경 통일 작업으로 이번 최적화 범위를 확대하지 마라.

### 2.2 두 번째 PC 접근이 안 되어도 로컬 결과를 완성한다

프로젝트에 이미 설정·승인된 원격 실행 경로가 있으면 사용한다. 없는 주소·계정·자격 증명을 추측하거나 새 연결을 설정하지 마라.

두 번째 PC에 접근할 수 없으면 현재 PC의 구현·측정을 끝내고, 같은 코드와 입력으로 두 번째 PC에서 실행할 정확한 명령을 `RUN_ON_SECOND_PC.md`에 남겨라. 결과 폴더 형식과 취합 명령도 포함한다.

이 경우 두 번째 장치를 `not_run_remote_unavailable`로 표시한다. 로컬 결과를 복사하거나 하드웨어 사양에서 추정하여 두 GPU 실측으로 표시하지 마라. 최종 상태는 `partial_local_complete`로 구분한다.

두 환경의 compiled kernel cache, graph 객체, cuDSS handle/factor의 메모리 이미지를 복사하여 공유하지 마라. 공유 대상은 공통 소스·설정·직렬화 가능한 원본 입력과 측정 결과다.

## 3. 단계 B — 동일 문제의 FP64 baseline 재현

이전 프로파일과 동일한 체크포인트·mesh·외력 시간·물리 구간·검산 설정을 우선 사용한다. 기존 로그의 backend 이름만 보고 Gauss를 강제하지 말고 실제 Newmark/Gauss/retry 사용 횟수를 기록하라.

각 PC는 자기 자신의 변경 전 실행을 baseline으로 가진다. 과거 5070의 약 1.55초나 과거 두 PC의 약 1.2배 차이를 이번 성공 기준으로 강제하지 마라.

각 반복 전에 위치뿐 아니라 속도·가속도·hi/lo·외력 상태·RNG·전처리기/캐시 초기화 정책을 동일하게 복원한다. GPU별 전처리된 기하·mesh·구적 데이터가 달라지지 않도록 입력 해시와 배열 순서를 확인한다. 관련 데이터를 비교하려고 새 전처리 알고리즘을 만들지는 마라.

일반 실행은 baseline과 후보 각각 최소 3회 측정한다. 후보 앞뒤에도 baseline을 배치하여 시간 변화·워밍업 영향을 확인한다. 후보 순서는 균형 있게 바꾸거나 고정 seed로 섞어 기록한다. 재현 가능한 차이가 없으면 최적이라고 단정하지 않는다.

같은 GPU에서는 측정 작업을 동시에 여러 개 실행하지 않는다. 서로 다른 PC의 독립 실행은 가능하다. 다른 GPU 작업이나 온도·클록 상태는 확인 가능한 범위에서 기록하되 시스템 설정을 바꾸지는 않는다.

### 시간 계측 계약

| 시간 | 정의 |
|---|---|
| setup_s | 컴파일·최초 할당·워밍업·초기 분석·Graph 준비 등의 별도 비용 |
| solver_s | 지정 물리 구간 solver의 계측 범위와 clock 종류를 명시한 시간 |
| audit_s | 독립 검산 시간. solver 내부 잔차와 구분 |
| transfer_s / save_s | 실제 전송·저장 비용. 수행하지 않으면 not_measured/not_applicable |
| compute_audit_wall_s | solver+audit 전체 구간을 관련 GPU 작업 완료까지 직접 측정한 wall-clock |
| process_wall_s | 초기화·전송·저장 등을 포함한 프로세스 전체 시간 |

solver와 audit에 별도 NVTX 범위를 두되, 모든 커널 뒤에 동기화를 넣지 마라. 기존 Graph를 깨거나 구간의 실행 구조를 바꿔야만 세분 시간이 얻어지면, 그 결과를 별도 진단 실행으로 분리한다. 분리 측정이 불가능한 항목은 합산 범위만 정확하게 보고한다.

CUDA event에 의한 GPU 시간과 CPU wall-clock, inclusive 시간과 exclusive 시간을 구분하라. 중첩 시간·CPU 대기·GPU 커널 duration 합을 중복 합산하지 않는다. 프로파일러를 붙인 실행의 시간을 일반 실행 가속률로 사용하지 않는다. [S2]

Newton/GMRES, HVP 호출, factorization/rebuild, fallback, retry, line-search backtrack은 실제 계측값만 남긴다. 총시간이나 다른 평균에서 횟수를 추정하지 마라.

## 4. 단계 C — GPU별 FP64 block_dim sweep

### 4.1 변경 대상을 좁힌다

현재 저장소의 force 경로에서 `precision.volume_kernel`과 `precision.edge_kernel`의 실제 launch를 찾는다. 참고 보고서에서는 `p3_shell_resident.py` 호출부였으나 현재 파일 위치가 바뀌었을 수 있다.

우선 volume, interior edge, boundary edge 세 역할을 구분한다. 전역 block_dim을 바꿔 HVP·GMRES·다른 reduction까지 한꺼번에 바꾸지 마라.

각 GPU에서 독립적으로 다음을 측정한다.

```text
block_dim 후보: 32, 64, 128, 256
대상 정밀도: 현재 FP64 hi/lo
대상 역할: volume / interior_edge / boundary_edge
```

소스 본문, 논리 작업 크기와 thread-to-work 매핑의 의미는 유지한다. 블록 크기가 바뀌어도 모든 유효 (element, quadrature) 또는 (edge, quadrature) 항목이 정확히 한 번 처리되는지 확인한다. boundary 조건, padding, warp 단위 처리 가정이 발견되면 안전성부터 검사한다.

실제 컴파일러의 block specialization으로 바이너리가 달라질 수는 있으므로 컴파일 옵션과 변형 식별자를 남긴다. 동일 수치식을 유지하되 바이너리 동일성까지 강제하지 않는다.

### 4.2 먼저 고정 작업량, 이후 실제 구간

isolated harness에서 동일한 원본 입력과 호출 수로 커널별 후보를 측정한다. 충분히 워밍업하고, 반복 호출을 CUDA event로 묶어 계측한다. 후보 간 호출 수는 동일하게 한다.

직접 덮어쓰는 출력과 누적하는 출력을 구분하여 reset 필요성을 확인한다. 반복 호출로 버퍼가 오염되어 다른 작업이 되지 않게 한다. reset·변환 시간을 분리할 수 없다면 포함 범위를 명시하고, 임의로 차감하여 음수 시간 등을 만들지 마라.

같은 작은 입력을 반복하는 cache-hot 시험이라는 한계를 명시한다. 이를 실제 궤적의 전체 가속률로 해석하지 마라.

커널별 유망 설정을 선택한 뒤 조합하여 원래 solver+audit 구간에서 검증한다. 처음부터 모든 블록 조합을 전수 조사하지 않는다. 후보가 개선되지 않으면 baseline 선택도 정상적인 결과다.

Graph는 설정 결정 후 각 PC에서 생성한다. 기존 256 설정으로 캡처한 Graph를 그대로 재사용하고 블록만 바뀌었다고 기록하지 마라. 실제 block/grid와 적용 role을 확인한다.

독립 검산은 별도 상태·잔차·판정 경로를 유지한다. solver의 힘 버퍼나 캐시를 재사용하여 검산을 생략/공유하지 않는다. audit가 같은 대상 커널을 호출한다면 launch 설정 적용 여부를 명시하되, 방정식·정밀도·검산 독립성은 바꾸지 않는다.

기존 검산과 회귀 검사를 통과한 후보만 유효 후보로 분류한다. bitwise equality는 관측 항목이지 새 필수 조건이 아니다. 반대로 bitwise 차이를 숨기려고 허용오차를 풀지도 않는다. 기존 기준으로 판정할 수 없는 차이는 `decision_required`로 남겨라.

## 5. 단계 D — 최소 GPU별 튜닝/캐시 구조

복잡한 범용 autotuner는 만들지 마라. 공통 설정 조회 함수, 소규모 sweep, JSON 결과 저장·조회 정도로 구성한다. 기존 harness/CLI가 있으면 확장한다.

### 5.1 실행 모드

다음 의미의 모드를 제공하되 옵션 이름은 현재 CLI에 맞춰 정한다.

| 모드 | 동작 |
|---|---|
| baseline/off | 기존 launch 설정. 코드의 생산 기본 동작은 계속 이것으로 유지 |
| explicit | 실험자가 지정한 커널별 block_dim 사용. 후보임을 결과에 표시 |
| cached | 현재 조건과 정확히 일치하는 검증된 프로파일 사용. miss/stale/invalid면 기존 설정으로 복귀하고 이유 기록 |
| tune | 명시적으로 sweep·검증 수행 후 별도 실험 프로파일 작성. 성공 시 해당 실험 실행에서만 사용 가능 |

옵션 우선순위와 충돌 처리를 명확히 한다. 기본값을 몰래 `tune`이나 `cached`로 바꾸지 않는다. 최초 자동 튜닝도 명시적으로 opt-in한 실행의 준비 단계에서만 수행한다.

타임스텝 도중이나 CUDA Graph capture 내부에서 benchmark·캐시 선택·재튜닝을 실행하지 않는다. profile은 solver/audit Graph 생성 전에 확정하고, 실행 중 고정한다.

### 5.2 캐시 키와 검증 범위

GPU 이름만으로 키를 만들지 않는다. 최소 다음을 구분한다.

```text
device_signature:
  실제 GPU 모델/UUID, compute capability, SM 수
build_signature:
  공통 수치 커널 및 의존 함수의 source hash
  실제 compiler/runtime/Warp 및 관련 라이브러리 버전
  컴파일 옵션, precision variant
workload_signature:
  정확한 mesh/topology 식별자
  node/element/interior-edge/boundary-edge 수
  각 kernel의 quadrature 수와 실제 launch logical shape
  입력 dtype/layout, kernel role, 실행 backend/Graph mode
validation_scope:
  checkpoint/case 식별자, 검증한 물리 구간, 검산 계약 식별자
```

이번에는 대표 mesh의 exact-match 프로파일부터 만든다. small/medium/large로 임의 구간을 만들고 미측정 mesh에 같은 설정을 자동 적용하지 마라. 여러 크기에 대한 일반화는 후속 작업이다.

설정이 달라지거나 코드·환경·작업 signature가 달라지면 기존 profile을 `stale`로 처리한다. 다른 GPU의 profile을 이름만 바꿔 사용하지 마라.

프로파일에는 최소 다음 정보가 있어야 한다.

```text
schema_version, profile_id, device/build/workload signature
volume_block_dim, interior_edge_block_dim, boundary_edge_block_dim
solver/audit 적용 범위
baseline과 후보의 원시 측정 결과 위치와 요약 통계
local_audit_status, regression_status, validation_scope
cross_device_status
precision_policy = unchanged_fp64_baseline
production_enabled = false
```

값은 실측 후 기록한다. 예제 최적값을 미리 넣지 않는다. 실패 후보나 FP32 microbenchmark 결과를 FP64 검증 프로파일로 등록하지 않는다.

캐시는 임시 파일 작성 후 atomic rename 등으로 완성된 파일만 보이게 저장한다. 알 수 없는 schema·부족한 필드·허용되지 않은 block_dim이면 실행에 적용하지 말고 기존 설정으로 복귀한다. worker별 출력 경로를 분리해 충돌을 막는다.

## 6. 단계 E — 각 GPU의 기존 FP32 fixed-work 비교

동일한 FP64 원본 체크포인트에서 현재 구현의 변환 함수를 이용한다. FP32 hi/lo를 우선 측정하고, 단일 FP32 경로가 이미 있으면 별도로 기록한다. 단순 dtype cast가 기존 보정된 FP32 경로를 대체해서는 안 된다.

현재 256 설정과 해당 GPU의 FP64 유망 설정에서 비교한다. FP32에서 최적 블록이 다를 수 있으므로, 기존 동일 harness로 안전하게 실행 가능하면 FP32도 같은 네 후보를 짧게 sweep한다. 그렇지 않으면 측정한 두 설정만 명시하고 FP32 최적값을 찾았다고 주장하지 않는다.

다음 두 효과를 따로 제시한다.

1. 동일 block_dim에서 FP64 → FP32 변화: 해당 실행 구성에서 정밀도 경로를 바꾸는 효과.
2. 각 정밀도에서 측정된 최선 block_dim끼리 비교: 정밀도 변화와 launch 튜닝이 합쳐진 효과.

필수 출력은 커널 GPU 시간·호출 수·변환/reset 비용·NaN/Inf·기하량과 조립 힘의 오차다. 상태 저장, 기하·법선·변형률, 구성식/gradient, 누적·조립의 실제 정밀도를 `precision_map.json`에 기록한다.

배열 dtype만 FP32인데 내부 계산은 FP64라면 순수 FP32라고 표시하지 않는다. FP32 hi/lo도 단일 FP32와 별도 variant다.

FP32 전체 적분 완주와 엄격한 teacher 검산 통과는 이 단계의 요구사항이 아니다. 검산에서 실패할 수 있는 출력도 finite하고 수행 작업이 명확하면 microbenchmark 자료로 남길 수 있지만, 검증된 teacher나 생산용 profile로 승격하지 않는다.

기존 FP32 경로에 최소 어댑터를 연결하는 것은 허용한다. 없는 solver·수치 알고리즘을 구현하지 않는다. 경로를 안전하게 연결하지 못하면 `not_measured`와 실제 원인을 남긴다.

## 7. 단계 F — 두 PC의 결과 비교

### 7.1 비교 행렬

| 비교 | 목적 |
|---|---|
| 1080 Ti baseline vs 1080 Ti tuned FP64 | 그 PC에서의 최적화 효과와 수치 영향 |
| 5070 baseline vs 5070 tuned FP64 | 그 PC에서의 최적화 효과와 수치 영향 |
| 1080 Ti baseline vs 5070 baseline | 최적화 전 PC/환경 차이 |
| 1080 Ti tuned FP64 vs 5070 tuned FP64 | 각자 최적화한 뒤의 PC/환경 차이 |

같은 문제·입력 해시·배열 대응·물리 시각으로 비교한다. backend나 중요한 환경이 달라지면 `matched_backend=false`와 차이를 남긴다. 비교 가능한 kernel-only 결과는 별도로 제시할 수 있지만, 다른 알고리즘의 전체 시간 비율을 같은 solver의 GPU 가속이라고 보고하지 않는다.

CPU가 다른 두 PC의 wall-clock 비율은 PC+실행 환경 비교다. GPU 자체만의 차이로 부르지 않는다.

### 7.2 상태·힘·에너지 비교 방법

각 PC에서 기존 독립 검산을 각각 실행한다. 짧은 구간의 시작·중간·끝 등 공통 출력 시각을 지정하여 위치·속도·힘·에너지/장부 항목을 비교한다. 가능하면 기존 소형 snapshot 출력 기능을 이용한다.

adaptive retry로 내부 시간 격자가 달라졌다면 스텝 번호만 맞추어 비교하지 않는다. 동일 물리 시각의 공통 출력이 없는 항목은 비교 불가로 표시한다. 새 보간 방법을 만들어 오차를 숨기지 마라.

hi/lo 상태는 보존하여 내보낸다. 차이를 계산하기 전에 단일 FP32로 합치지 않는다. 기존 보상 차분 루틴이 있으면 사용하고, 최종 비교의 누적 정밀도·정규화 방법을 기록한다.

모든 비교는 절대오차를 우선 기록한다. 공통 free DOF 집합 F에서 각 성분 a의 차이를 d_a(t)=z_a^A(t)-z_a^B(t), D=|F|라 하면:

\[
 e_{z,\mathrm{rms}}(t)=\sqrt{\frac{1}{D}\sum_{a\in F}d_a(t)^2},
 \qquad
 e_{z,\infty}(t)=\max_{a\in F}|d_a(t)|.
\]

z=q이면 단위는 m, z=v이면 m/s, z=f이면 N이다. 전 성분 RMS인지 노드별 3D 벡터 norm인지 이름에 명시하고 혼합하지 않는다. D=0인 역할은 `not_applicable`이다.

위치의 상대오차 분모로 원점에서의 절대 위치 norm을 사용하지 않는다. 기존 프로젝트에 명시된 양의 길이 척도 L_ref가 있으면 e_q,rms/L_ref를 추가한다. 동일하게 기존 v_ref, F_ref, E_ref가 양의 물리 척도로 정의되어 있을 때만 정규화한다. 정의가 없으면 새 허용오차를 만들어 채우지 말고 절대오차와 `scale_not_defined`를 남겨라.

microbenchmark의 비영 기준 배열 z_ref에는 다음 진단 상대오차를 추가할 수 있다.

\[
 e_{z,\mathrm{rel2}}=
 \frac{\sqrt{\sum_a(z_a-z_{a,\mathrm{ref}})^2}}
 {\sqrt{\sum_a z_{a,\mathrm{ref}}^2}}.
\]

분모가 0이면 relative 값은 null로 두고 절대오차를 보고한다. 이 진단량 자체를 새로운 합격 기준으로 사용하지 않는다.

에너지는 기존 코드가 정의한 각 항의 절대 차이 |E^A(t)-E^B(t)| [J]와 기존 장부 검사값을 기록한다. Newmark의 물리적 에너지 오차가 정확히 0이어야 한다는 새 요구를 추가하지 않는다.

구간 내 오차는 표본별 원시값과 최대값을 남긴다. 힘은 동일 입력 fixed-state의 차이와, 서로 다른 궤적에서 발생한 힘 차이를 구분한다.

### 7.3 합격 기준을 새로 만들지 않는다

두 PC가 각각 기존 검산에 통과했다는 사실과, 학습 label의 PC 간 동일성이 충분하다는 판정은 분리한다. 프로젝트에 교차 장치 오차 예산이 있으면 그대로 적용한다. 없다면 차이를 정량 보고하고 `cross_device_acceptance=budget_not_defined`로 표시한다.

짧은 한 구간만 검증하고 장기 안정성·전체 dataset 병합 가능성을 확정하지 않는다. cross-device 예산이 미정이어도 각 PC의 로컬 성능 시험 결과는 완성할 수 있다.

## 8. 프로파일러는 보조 진단이다

5070에서는 설치된 도구가 지원하면 baseline과 최선 FP64 후보의 상위 두 커널을 제한적으로 NCU로 비교한다. block/grid, registers, occupancy, FP64 active/elapsed, 수집 가능한 spill 지표를 기록한다.

conditional Graph 때문에 수집이 막히면 실제 입력을 사용하는 isolated 실행을 진단용으로 이용한다. 진단 실행을 생산 실행과 같다고 표시하지 않는다.

현재 Nsight Compute 지원 표에는 Pascal이 지원되지 않는 것으로 명시되어 있다. 1080 Ti에 같은 NCU 측정을 강제하지 마라. 해당 PC에서 가능한 CUDA event·wall-clock·실제 launch metadata를 먼저 완성하고, 다른 프로파일러는 이미 설치되어 있고 지원될 때만 사용한다. [S3]

도구 설치·업그레이드·권한 변경으로 범위를 늘리지 않는다. 수집 불가 지표는 not_measured이고, 장치 사양으로 만든 추정값을 실측 칸에 넣지 않는다.

## 9. 통계와 가속률 계산

각 반복의 원시 시간을 보존한다. 일반 실행 중앙값은 다음과 같다.

\[
 \widetilde T=\operatorname{median}(T_1,\ldots,T_R),
 \qquad
 \operatorname{MAD}(T)=\operatorname{median}_r|T_r-\widetilde T|.
\]

R은 유효 반복 수이며 최소 3회다. min/max와 MAD도 함께 제시한다. 차이가 측정 변동과 비슷하면 tie/inconclusive로 남기고 baseline을 유지한다. 소수 반복으로 통계적 유의성이나 성공 확률을 만들어내지 않는다.

각 PC g의 solver+audit 가속률과 시간 감소율은:

\[
 S_g=\frac{\widetilde T_{g,\mathrm{baseline}}}
 {\widetilde T_{g,\mathrm{candidate}}},
 \qquad
 r_g=1-\frac{\widetilde T_{g,\mathrm{candidate}}}
 {\widetilde T_{g,\mathrm{baseline}}}.
\]

T는 같은 물리 구간의 직접 측정한 compute_audit_wall_s이다. r_g의 백분율은 100*r_g이며, S_g-1과 혼동하지 않는다. 실패/미완료/검산 불통과 실행은 유효 가속률 계산에서 제외하되 실패 비용은 삭제하지 않는다.

고정 작업량에서 반복 r의 C회 호출 GPU 시간을 T_gpu,r이라 하면 호출당 시간 t_r=T_gpu,r/C이다. 같은 커널·입력·호출 정책에서 med(t_FP64)/med(t_FP32)를 계산한다. reset·변환 포함 여부가 다르면 그 비율을 직접 비교하지 않는다.

두 PC의 비율은 같은 설정 비교와 각 PC의 최선 설정 비교를 구분한다. 단순 TFLOPS 비율·과거의 1.2배·Amdahl 가정값은 실측 결과를 대신하지 않는다.

## 10. 필수 산출물

기존 포맷을 재사용하되, 모든 원시 행이 어느 PC·환경·case·variant에 해당하는지 추적 가능하게 만든다.

```text
results/<experiment_id>/
  report.md
  experiment_manifest.json
  source_changes.diff
  changed_files.txt
  reproduce.md
  RUN_ON_SECOND_PC.md                 # 원격 미실행 또는 수동 재현이 필요할 때
  workers/<worker_id>/
    environment.json
    compatibility.json
    case.json
    precision_map.json
    run_summary.csv
    step_stats.csv
    block_sweep.csv
    fixed_work.csv
    output_errors.csv
    selected_launch_profile.json
    status.json
    snapshots/                       # 비교에 필요한 소형 원본 배열/hi·lo
    profiler/                        # 수행한 경우 CSV·텍스트 export
    logs/
  comparison/
    cross_device_metrics.csv
    comparison_status.json
```

비어 있는 파일을 성공 산출물로 세지 않는다. 미실행 항목은 status/reason을 제공하고, 읽는 코드가 누락/null/not_applicable을 처리하도록 한다.

### 원시 CSV 최소 열

공통 식별자는 experiment_id, run_id, worker_id, device_signature, build_signature, case_hash, variant_id, repeat_id, status, reason이다. 긴 signature는 별도 manifest를 참조하는 ID로 줄여도 된다.

- `run_summary.csv`: 물리 시작·종료 시각, backend/graph mode, 실제 profile, setup/process/compute_audit 시간, 분리 가능한 solver/audit 시간과 clock 정의, completed, audit_pass, 반복·retry·fallback 요약.
- `step_stats.csv`: 물리 시각·dt, actual integrator, Newton/GMRES와 계측 가능한 호출 횟수, line search/rebuild/retry/fallback, 기존 검산 필드. 각 필드 정의와 단위를 별도 문서에 둔다.
- `block_sweep.csv`, `fixed_work.csv`: kernel/role, precision variant, logical shape, block/grid, 호출 수, GPU 시간, reset/변환 시간과 범위, output 오류 결과 연결 ID.
- `output_errors.csv`: 비교 쌍·같은 입력 여부·물리 시각·관측량·norm·단위·참조 scale·절대/상대 오차·NaN/Inf·기존 기준 통과 여부.

선택 profile과 모든 표의 값은 원시 CSV까지 추적되어야 한다. 명령에는 실제 사용한 CLI 옵션과 환경을 남긴다. 명세에만 있는 가상의 옵션을 실제 실행했다고 기록하지 마라.

## 11. 최소 테스트와 중단 조건

필수 테스트는 기존 회귀·검산에 더하여 설정 선택 부분만 최소로 추가한다.

- 옵션을 주지 않으면 기존 launch가 유지되는가?
- 캐시 hit/miss/stale/잘못된 schema에서 올바른 설정 또는 기존 설정을 쓰는가?
- 다른 GPU·precision·mesh·build의 profile이 잘못 적용되지 않는가?
- volume/interior/boundary 설정이 실제 launch/Graph에 반영되는가?
- benchmark 후 상태와 누적 버퍼가 기준대로 복원되는가?

같은 원인으로 수정·재실행이 두 번 연속 실패하면 해당 후보/도구 경로는 중단하고 원인·명령·로그를 남긴다. 생산 소스를 반복 손상시키거나 허용오차를 완화해 통과시키지 마라.

성능 개선이 없거나 한 GPU에서만 개선되어도 유효한 결과다. 개선되지 않는 장치는 baseline을 유지한다. 두 장치 모두 동일한 가속률을 얻는 것을 완료 조건으로 삼지 않는다.

## 12. 최종 report.md 첫머리에 답할 내용

1. 각 PC의 실제 환경·접근/호환성 상태와 baseline 재현 여부.
2. GPU별 커널 최선 block_dim, 그리고 기존 검산을 통과한 solver+audit 전체 가속률·시간 감소율.
3. GPU별 FP32 fixed-work 가속률과 출력 오차. 동일 블록/측정 최선 블록 비교를 분리.
4. 공통 코드 + GPU별 profile 선택/캐시가 실제 작동했는지와 생산 기본값 유지 여부.
5. 두 PC의 교차 비교 결과, 미정확도 예산, 환경 차이와 검증 구간의 한계.
6. 다음 작업을 FP64 유지 최적화, FP32 정확도 개선, 잔여 병목 진단 중 어디에 두는 것이 실측상 타당한지.

모든 결과를 하나의 ZIP으로 묶는다. 두 번째 PC를 실행하지 못한 경우 로컬 결과 ZIP과 두 번째 PC 재현/취합 명령을 제공하고 미완료 상태를 명시한다. 데이터 생성 worker 분산 운영이나 장기 검증을 이번 단계에서 추가 구현하지 않는다.

## 참고: 공식 문서

이 문서의 실험·캐시·오차 보고 계약은 이번 프로젝트를 위한 지시이며, 아래 자료의 문구를 그대로 구현하라는 뜻은 아니다. 설치 버전의 실제 동작을 우선 확인한다.

[S1] NVIDIA CUDA Toolkit 13.0 Release Notes, §2.6.2 Deprecated Architectures. Pascal 등의 offline compilation/library 지원 제거와 CUDA 12.x 빌드 지원을 설명한다.

```text
https://docs.nvidia.com/cuda/archive/13.0.0/cuda-toolkit-release-notes/index.html#deprecated-architectures
```

[S2] NVIDIA CUDA C++ Best Practices Guide. 비동기 실행 시간 측정, CUDA event, 실행 구성과 레지스터/occupancy 절충에 관한 공식 지침.

```text
https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html
```

[S3] NVIDIA Nsight Compute Release Notes, GPU Support. Pascal 지원 여부와 버전별 Graph profiling 지원은 설치 버전에 맞춰 확인한다.

```text
https://docs.nvidia.com/nsight-compute/ReleaseNotes/index.html#gpu-support
```

[S4] NVIDIA cuDSS Release Notes 및 Getting Started. cuDSS는 toolkit 이름만으로 지원/동작을 가정하지 말고 실제 설치 버전·설정·실행 경로로 확인한다.

```text
https://docs.nvidia.com/cuda/cudss/release_notes.html
https://docs.nvidia.com/cuda/cudss/getting_started.html
```

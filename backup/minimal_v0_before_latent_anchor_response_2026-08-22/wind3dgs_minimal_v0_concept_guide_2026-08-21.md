# Wind3DGS Minimal V0 개념 해설 및 검토 가이드

## 이 문서의 목적과 권한

이 문서는 현재 아이디어 스케치를 내가 이해하고 비판적으로 검토하기 위한 개념 설명서다.
수식의 유도나 구현용 세부 규격보다, 각 구성요소가 왜 필요하고 서로 어떻게 연결되는지를 설명한다.

이 문서는 새로운 방법 계약을 만들지 않으며 canonical source를 대체하지 않는다. Method equation과
claim의 최종 권위는 canonical sketch가 소유하고, implementation checklist는 그 stable label을 참조하는
파생 실행 문서다. 이 해설과 두 문서가 충돌하면 그 순서에 따라 판단한다.

- [Canonical idea sketch](3dgs_topology_distilled_selective_global_local_wind_dynamics_2026-08-15.tex)
- [Implementation checklist](implementation_checklist_topology_distilled_selective_global_local_wind_dynamics_2026-08-15.tex)

현재 상태도 구분해야 한다. 지금 정리된 것은 구현 가능한 설계와 검증 계획이지, MC1·MC2·SYS가
실험으로 이미 입증되었다는 결과 보고서가 아니다.

## 1. 1분 요약

Static 3DGS는 물체의 색과 모양은 잘 표현하지만, 천의 어느 부분이 서로 이어져 있는지, 얼마나
무거운지, 어디가 고정되어 있는지, 힘이 어떻게 전달되는지는 알려주지 않는다.

이 연구는 많은 Gaussian 아래에 소수의 물리 anchor와 sparse한 연결 구조를 만든다. 매 frame 모든
anchor에 바람의 힘을 계산하고, 물체 전체의 큰 움직임은 Global mode로 항상 표현한다. Global만으로
부족한 끝단 떨림이나 국소 flutter는 현재 필요한 patch의 Local mode만 선택해 보충한다. Global과
Local을 따로 계산하지 않고 하나의 작은 coupled system에서 함께 푼 뒤, anchor 움직임을 원래의 많은
Gaussian에 전달해 렌더링한다.

핵심은 더 정교한 유체나 shell 물리를 만드는 데 있지 않다. 핵심 질문은 다음과 같다.

> 전체 움직임은 항상 유지하면서 현재 필요한 국소 움직임만 계산하면, 강한 Global 모델이나
> 모든 anchor를 직접 푸는 방법보다 더 좋은 시각 품질–실행시간 절충점을 만들 수 있는가?

## 2. 전체 흐름 한눈에 보기

```text
독립적으로 재구성된 static 3DGS
        │
        ▼
[Setup: asset마다 한 번]
소수 anchor 선택 → 표면 연결/정규화된 상대 면적 ownership → 재료·부착 적용
        │
        ├─ 늘어남·굽힘 구조와 질량/감쇠 package
        ├─ 전체 움직임용 Global basis
        ├─ patch별 complementary Local basis
        └─ anchor 움직임을 Gaussian에 전달할 binding
        │
        ▼
[Runtime: 매 frame]
이전 표면 → 모든 anchor의 바람 힘 → 필요한 Local 선택
        │
        ▼
Global + Active Local + Decay Local 단일 coupled solve
        │
        ▼
anchor 위치/속도 → Gaussian mean/covariance transport → 렌더링
```

Setup은 비교적 비싼 계산을 미리 해 두는 단계이고, runtime은 이미 만들어 둔 작은 package를 이용해
frame마다 빠르게 움직임을 계산하는 단계다.

### 자주 쓰는 말을 먼저 풀면

| 용어 | 이 문서에서의 쉬운 뜻 |
|---|---|
| Gaussian | 최종 화면의 위치·크기·방향·색을 표현하는 렌더링 요소 |
| Anchor | 많은 Gaussian의 움직임을 대신 계산하는 소수의 물리 조종점 |
| Scaffold | Anchor, 연결 관계, 면적·질량과 부착 조건을 합친 성긴 물리 골격 |
| Relation | 어느 anchor들이 늘어남 또는 굽힘을 함께 결정하는지 나타내는 연결 |
| Mode / basis | 물체가 움직일 수 있는 대표적인 변형 패턴과 그 패턴들의 모음 |
| Global | 물체 전체에 걸친 큰 움직임을 표현하는 mode |
| Local complement | 특정 patch에서 Global이 표현하지 못한 부분만 보충하는 mode |
| Local unit | 함께 선택하고 상태를 유지하는 Local mode 묶음 |
| Selector | 현재 solve에 넣을 Local unit을 정하는 결정론적 규칙 |
| Coupled solve | Global과 선택된 Local의 상호작용을 포함해 한 번에 푸는 계산 |
| Transport | Anchor 변형을 Gaussian mean과 covariance로 전달하는 과정 |
| p95 latency | 대부분의 frame을 포함하되 느린 꼬리 구간도 드러내는 95백분위 실행시간 |
| Pareto improvement | 같은 속도에서 품질이 좋거나, 같은 품질에서 더 빠른 새로운 절충점 |
| Source/Oracle scaffold | 배포 입력이 아니라 방법의 필요성과 예측 scaffold를 비교하기 위한 privileged 기준 골격 |

## 3. 왜 sparse anchor가 필요한가

Gaussian은 렌더링에는 좋지만, Gaussian 하나하나를 모두 물리 입자로 사용하면 다음 문제가 생긴다.

- Gaussian 수가 많아 물리 계산이 비싸다.
- 가까운 Gaussian이 실제로 같은 표면의 이웃인지 알기 어렵다.
- 얇은 물체에서는 앞면과 뒷면이 공간상 가까워 잘못 연결되기 쉽다.
- 재구성마다 Gaussian 밀도와 위치가 달라 같은 물체에도 물리 결과가 흔들릴 수 있다.

그래서 렌더링용 Gaussian과 시뮬레이션용 anchor의 해상도를 분리한다. Anchor는 많은 Gaussian 아래에
숨겨 놓은 성긴 천 골격과 비슷하다. 각 anchor는 다음 정보를 가진다.

- 위치와 국소 표면 방향
- 같은 표면층의 이웃 관계
- 담당하는 상대적 표면 면적과 질량
- 고정된 가장자리인지 여부
- Global/Local 움직임이 anchor에 미치는 영향

물리 계산량은 주로 anchor와 선택된 mode 수에 따라 결정되고, 최종 외관의 해상도는 Gaussian 수에
따라 결정된다. 이 분리가 SYS scaling 가설의 기반이다.

## 4. 움직임을 만들기 위해 필요한 힘과 효과

사용자 관점에서는 A, B, C의 세 묶음으로 이해할 수 있다. 다만 정확히는 세 개의 독립적인 외력이
아니라 외부 자극, 내부 복원, 운동 응답을 나눈 것이다.

| 묶음 | 역할 | 개념적인 계산 재료 | 화면에서 보이는 결과 |
|---|---|---|---|
| A. 바람의 외력 | 물체에 에너지를 넣고 움직이게 한다 | 바람과 표면 속도의 차이, 표면 법선, 담당 면적, 공기 밀도와 계수 | 깃발이 바람 방향으로 밀리고 flutter가 시작됨 |
| B. 구조 복원 효과 | 늘어나거나 굽은 표면을 원래 형태 쪽으로 되돌린다 | 이웃 anchor의 원래 거리와 현재 거리, 한 anchor와 양옆 이웃의 굽힘 관계, 재료 강성 | 천이 무한히 늘어나지 않고 휘었다가 돌아옴 |
| C. 관성과 감쇠 | 움직임을 이어 가게 하면서 진동이 영원히 남지 않게 한다 | anchor 면적 기반 질량, 이전 속도, 고정된 damping preset | 바람이 멈춰도 잠시 흔들리고 점차 안정됨 |

Hard attachment는 네 번째 힘이라기보다 경계 조건이다. 깃발의 왼쪽 가장자리처럼 움직이지 않을
anchor의 자유도를 Setup에서 제거한다. Minimal V0는 그 지점의 정확한 support reaction을 복원하거나
공학적 힘으로 주장하지 않는다.

### A. 바람의 외력

각 anchor는 자기 위치의 바람과 자기 속도의 차이, 즉 상대 바람을 느낀다. 상대 바람을 표면에
수직인 성분과 표면을 따라가는 성분으로 나누어 정면 압력과 접선 방향 끌림을 근사한다. 담당 면적이
넓고 상대 풍속이 클수록 힘이 커진다.

이 힘은 매 frame 모든 anchor에서 한 번 계산한다. Local이 선택된 patch에만 바람을 계산하는 것이
아니다. 지나치게 큰 힘은 방향을 유지한 채 안전 범위로 제한하고, 제한이 자주 발생하면 정상적인
성공으로 숨기지 않고 failure/clamp 통계에 남긴다.

이 모델은 빠른 quasi-steady 근사다. 정확한 lift, wake, vortex shedding 또는 유체 이력을 푸는 CFD가
아니다.

### B. 구조 복원 효과

구조는 두 종류의 단순한 관계로 만든다.

- Stretch 관계는 연결된 두 anchor가 원래 거리에서 과도하게 늘어나거나 줄어들지 않게 한다.
- Bending 관계는 한 anchor와 양옆 이웃이 얇은 표면답게 휘도록 하고, 과도한 국소 꺾임을 되돌린다.

굽힘 기준은 고정된 world 방향이 아니라 물체와 함께 회전하는 국소 표면 방향에서 정한다. 따라서
물체 전체가 회전한 것만으로 가짜 굽힘 복원 효과가 생기는 것을 줄인다.

Frame 시작에 이전 상태로 예상 위치를 만들고 stretch/bending이 향할 target을 한 번 정한다. 이
target과 구조의 Hessian은 해당 frame의 solve 동안 고정한다. 정확한 nonlinear shell을 반복해서 푸는
대신, 시각적으로 안정적인 한 번의 fixed-Hessian PD-style step을 사용한다.

### C. 관성과 감쇠

각 anchor의 질량은 담당 면적과 사용자가 지정한 surface density로 만든다. 면적은 engineering-grade
실측 면적이 아니라 재구성 밀도가 달라도 상대적인 움직임 scale을 일관되게 만들기 위한 deterministic
ownership convention이다.

질량은 주어진 외력과 구조 조건에서 가속도와 관성 응답을 정하고, 이전 속도와 함께 움직임이 얼마나
계속되는지를 결정한다. 감쇠는 남은 진동을 줄인다. 감쇠가 너무 크면 원하는 flutter까지 사라질 수
있으므로, zero-wind에서 안정적으로 rest로 돌아오면서 목표 주파수대의 움직임은 남는지를 함께 확인한다.

여기서 면적은 두 역할을 구분해야 한다. Setup에서 고정한 rest area ownership은 질량과 구조 weight를
만든다. Runtime의 deformation-aware safe area는 frame 시작 표면의 면적 변화율을 안전 범위로 제한한
값이며 aero만 소비한다. 제한 전 raw area ratio는 diagnostic과 clamp-event 기록용이지 aero 입력으로
직접 사용하지 않는다.

### 가장 단순한 직관

```text
바람이 민다
  + 구조가 원래 모양으로 되돌리려 한다
  + 질량이 반응 속도와 관성을 정한다
  + 감쇠가 남은 진동을 줄인다
  = 다음 frame의 움직임
```

Global과 Local은 위 힘들 자체가 아니다. 같은 힘에 반응할 수 있는 움직임의 표현 공간이다.

## 5. Setup에서 한 번 준비하는 것

### 5.1 입력

Setup에는 다음이 필요하다.

- Static Gaussian의 위치, covariance와 appearance
- 실제 SI scale 또는 명시적인 nondimensional scale
- Surface density, stretch/bending 강성, damping preset
- 공기 밀도와 quasi-steady force 계수
- 고정할 hard attachment anchor

재료와 부착 조건은 appearance에서 추측하지 않고 사용자가 지정한다.

### 5.2 Topology distillation

Static GS에는 triangle connectivity가 없다. 단순한 거리 기반 kNN은 얇은 물체의 앞면과 뒷면을
잘못 연결할 수 있다. Topology distillation은 Setup에서 다음 정보를 만든다.

- 두 anchor가 같은 표면의 유효한 이웃인지
- 가까워 보이지만 다른 층을 잇는 shortcut인지
- 각 anchor가 전체 면적 중 얼마를 담당하는지

Training 때 source mesh는 label을 만드는 privileged supervision으로 사용할 수 있다. 그러나 target
Setup과 runtime에는 target triangle mesh를 요구하지 않는다.

Network가 예측하는 범위도 제한되어 있다. Network는 관계 score, shortcut/layer score와 상대적 area
ownership을 만든다. 재료, attachment, 질량·강성 matrix, 바람의 힘, 동적 state 또는 움직임을 직접
예측하지 않는다. 예측 결과는 고정된 analytic decoder와 validity check를 거쳐 stretch/bending 구조와
질량 package로 변환된다.

### 5.3 Global과 Local 움직임 사전 계산

구조 package가 만들어지면 움직임의 작은 사전을 미리 만든다.

- Global basis는 전체 sway, 큰 휨과 낮은 주파수 응답을 담는다.
- Patch-local basis는 특정 영역의 국소 움직임을 담는다.
- Local basis에서는 Global이 이미 표현하는 부분을 제거해 complementary하게 만든다.

Local은 단순히 Global mode를 작은 영역에 복사한 것이 아니다. “전체 움직임으로 이미 가능한 것”을
제외하고, 해당 patch에서 추가로 필요한 움직임만 남기는 것이 핵심이다.

### 5.4 Gaussian binding

각 Gaussian은 같은 표면층의 주변 anchor와 미리 연결된다. 나중에 anchor들이 이동하면 그 주변의
국소 회전과 변형을 추정하여 Gaussian mean과 covariance를 갱신할 수 있다.

## 6. Global과 Local이 표현하는 거시적·국소적 움직임

| 표현 | 담당하는 움직임 | Runtime 상태 |
|---|---|---|
| Global | 깃발 전체의 sway, 큰 굽힘, 저주파 반응 | 항상 solve에 포함 |
| Local | 끝단 flutter, 국소 굽힘·진동, traveling gust가 지나가는 부위의 세부 반응 | 필요한 unit만 포함 |
| Active Local | Pre-solve admission을 통과한 실제 Local 집합 | 새 external excitation을 온전히 받음 |
| Decay Local | Active에서는 빠졌지만 움직임이 아직 남은 Local | 새 excitation만 줄고 관성·복원·감쇠는 유지 |

여기서 “국소적”은 Gaussian 하나의 미시 물리를 뜻하지 않는다. Sparse anchor patch 단위에서 Global이
놓치는 움직임을 보완한다는 뜻이다.

### 왜 Global rank만 계속 늘리지 않는가

Global rank를 늘리면 더 많은 움직임을 표현할 수 있지만, 한쪽 끝의 짧은 flutter를 위해 물체 전체의
계산 차원을 늘리게 된다. 이 연구의 가설은 전체 움직임용 Global은 적정 크기로 유지하고, 현재 필요한
지역의 Local만 잠깐 추가하는 편이 같은 실행시간에서 더 유리할 수 있다는 것이다.

이 가설은 아직 결과가 아니다. Strong Global-rank sweep과 같은 p95 dynamics latency에서 반드시 비교해야 한다.

### 왜 Global과 Local을 한 번에 푸는가

Global을 먼저 풀고 Local 보정을 따로 더하면 두 움직임 사이의 질량·강성·감쇠 상호작용을 놓칠 수
있다. 현재 방법은 Global, Active Local과 Decay Local을 하나의 combined basis로 묶고 모든 cross
interaction을 포함한 작은 system을 한 번 푼다.

같은 anchor 바람 힘이 Global 좌표와 Local 좌표 양쪽에 표현되어도 double counting이 아니다. 같은
anchor force의 virtual work를 combined basis의 각 coordinate로 표현하고, cross interaction을 포함한
한 system에서 푸는 것이다. Force를 Global용과 Local용으로 복제하거나 patch별로 잘라 재조립하지
않는다. Selector가 force를 읽어 점수를 계산하는 것도 두 번째 물리적 힘 적용이 아니다. 안전 범위를
통과한 external force field는 최종 coupled system에서 한 번만 소비된다.

## 7. 매 frame 실제로 일어나는 일

한 frame은 다음 순서로 진행된다.

1. 이전 Global/Local state에서 anchor 위치와 속도를 복원한다.
2. 이전 속도로 짧게 예측한 위치에서 그 frame의 stretch/bending target을 한 번 정한다.
3. Frame 시작 geometry에서 각 anchor의 표면 법선과 안전하게 제한된 면적을 계산한다.
4. 모든 anchor의 quasi-steady wind force를 한 번 계산한다.
5. 같은 force field를 이용해 Local unit의 필요성 score를 계산한다.
6. Minimum-dwell unit을 유지해 desired set을 만든 뒤, 재활성화·빈 slot admission·capacity-checked
   replacement를 거쳐 실제 Active/Decay solve 집합을 정한다.
7. Global과 Active+Decay Local을 하나의 coupled system으로 한 번 적분한다.
8. Solve 뒤 충분히 작아진 Decay state만 frame 끝에서 제거한다.
9. 최종 anchor 움직임을 Gaussian mean과 covariance에 전달하고 렌더링한다.

Prior state에서 pre-solve membership을 먼저 정하고 한 번 solve한 뒤, post-solve/pre-zero state로
release를 판정한다. 해제된 state는 frame 끝에서만 0으로 만들며 membership 제거는 다음 frame solve부터
적용한다. 같은 frame을 다시 풀지 않는다.

새로 움직인 geometry는 다음 frame의 바람 계산에 반영한다. 한 frame 안에서 바람을 다시 계산하거나
corrector를 수행하지 않는 의도적인 one-frame stagger다. 이것이 영상에서 보이는 phase lag나 불안정을
만들고 same-p95에서 실제 개선 가능성이 확인될 때만 same-frame corrector를 다시 검토한다.

## 8. Selector와 Active/Decay의 직관

### Selector가 실제로 고르는 것

Selector는 바람을 받을 anchor를 고르지 않는다. 바람은 이미 모든 anchor에서 계산되었다. Selector는
현재 frame의 coupled solve에 어떤 Local 움직임 어휘를 넣을지를 고른다.

각 Local unit에 대해 다음 두 가지를 결합한 저렴한 score를 쓴다.

- 현재 바람이 그 Local 움직임 방향을 얼마나 강하게 자극하는가
- 그 unit이 그 자극에 얼마나 쉽게 반응할 것으로 보이는가

이 score는 미래 error, 정확한 영상 품질 향상, Global/다른 Local과의 전체 결합을 예측하지 않는다.
Certified error estimator가 아니라 cached isolated compliance proxy다.

### 왜 Active에서 바로 제거하지 않는가

Local state를 선택에서 빠진 순간 0으로 만들면 변위와 속도가 갑자기 사라져 pop이 생긴다. 그래서
두 상태만 유지한다.

- Active는 새 바람 excitation을 온전히 받는다.
- Decay는 새 excitation만 서서히 줄이지만 기존 변위, 속도, 관성, 탄성, 감쇠와 coupling은 유지한다.

Fade가 끝나고 에너지, 변위, 속도와 제거 시 예상되는 화면 pop이 모두 충분히 작은 상태가 여러 frame
유지될 때만 slot을 0으로 만든다. Decay 중 다시 필요해지면 state reset 없이 Active로 복귀한다.

설정상 Active capacity를 고정해도 valid candidate 수, Decay 수와 unit별 mode 수 때문에 실제 solve
rank와 p95 latency는 달라질 수 있다. 따라서 fixed-cardinality라는 말만으로 고정 비용을 주장하지
않고 실제 rank와 latency를 기록한다.

## 9. Anchor 움직임을 Gaussian에 전달하는 방법

물리 state는 anchor에만 있다. 각 Gaussian은 주변 anchor가 어떻게 이동하고 회전하고 늘어났는지를
보고 국소적인 surface transform을 맞춘다.

- Gaussian mean은 주변 anchor 움직임을 따라 이동한다.
- Gaussian covariance는 국소 회전과 변형에 맞춰 갱신한다.
- 정상 범위를 벗어난 scale과 covariance만 제한하고 event rate를 기록한다.

국소 affine fit이 불안정해도 표면 움직임을 안정적으로 rigid하게 설명할 수 있으면 renderer에만
proper-rigid fallback을 제한적으로 허용한다. 이 fallback은 바람이나 구조 계산의 normal/area를 만드는
물리 fallback으로 사용하지 않는다. 물리 표면이 불안정하면 canonical attempt는 failure로 남긴다.

정량 core에서는 opacity와 degree-0/DC color를 유지하고 higher-order view-dependent appearance는
비활성화한다. Minimal V0가 직접 갱신하는 렌더링 정보는 Gaussian mean과 covariance다.

## 10. 무엇이 학습되고 무엇이 deterministic한가

| 항목 | 담당 방식 |
|---|---|
| Anchor sampler와 candidate 생성 | Versioned Setup rule |
| 같은 층 relation, shortcut/layer score | Setup-only topology model |
| 상대적 area ownership | Setup-only topology model |
| 재료와 hard attachment | 사용자 입력 |
| Stretch/bending constraint, 질량·감쇠 | Analytic decoder |
| Global/Local basis | Offline deterministic solve |
| Wind force | Runtime analytic model |
| Local 선택 | Runtime deterministic score와 fixed policy |
| Global+Local dynamics | Runtime single coupled solve |
| Gaussian mean/covariance | Deterministic affine transport |

따라서 이 방법은 end-to-end neural simulator가 아니다. 학습은 static GS에서 쓸 수 있는 sparse physical
scaffold를 만드는 데만 사용하고, runtime dynamics는 해석적이고 결정론적으로 유지한다.

## 11. 깃발 예시로 이해하기

### Setup

왼쪽 가장자리가 고정된 rectangular flag의 static GS가 있다고 하자.

1. Gaussian보다 적은 anchor를 표면에 배치한다.
2. 앞면과 뒷면을 잘못 잇지 않도록 같은 층 relation을 만든다.
3. Edge와 opposite triplet으로 stretch/bending 구조를 만든다.
4. 왼쪽 edge를 hard attachment로 고정한다.
5. 전체가 휘는 Global mode와 끝단·중앙 patch의 Local mode를 미리 만든다.
6. 각 Gaussian을 주변 anchor에 binding한다.

### 일정한 바람

- 모든 anchor에서 상대 바람에 따른 힘을 평가하며, 상대 바람과 표면 방향에 따라 0일 수도 있다.
- Global은 깃발 전체가 뒤로 젖혀지고 천천히 흔들리는 움직임을 만든다.
- 끝단처럼 Local score가 높은 patch가 Active가 되어 빠른 flutter를 보충한다.
- 둘은 같은 coupled solve 안에서 서로 영향을 주며 결정된다.

### 돌풍이 왼쪽에서 오른쪽으로 이동

- 의도대로 작동한다면 돌풍 위치에 따라 높은 score를 받는 Local patch가 이동한다.
- Active slot과 Decay capacity에 여유가 있으면 새 patch가 admit된다. 여유가 없으면 activation이
  지연되고 그 지연을 측정한다.
- 지나간 patch는 즉시 사라지지 않고 Decay로 들어가 잔류 진동을 자연스럽게 줄인다.
- Global sway는 계속 유지된다.

### 바람이 갑자기 멈춤

- 외부 바람장은 0이 되지만 깃발이 아직 움직이면 상대 바람에 의한 aerodynamic drag는 잠시 남을 수 있다.
- 관성과 stretch/bending 복원으로 운동이 이어진다.
- Aerodynamic drag와 structural damping이 남은 에너지를 줄여 rest로 돌아가게 한다.
- 충분히 작아진 Local state만 안전하게 release된다.

## 12. 이 연구가 주장하려는 세 가지

| 기여 | 쉬운 설명 | 필요한 증거 |
|---|---|---|
| MC1: topology-distilled scaffold | Target mesh 없이 static GS에서 물리에 쓸 sparse scaffold를 만든다 | 서로 다른 independent GS에서도 안정적이고 source/Oracle scaffold에 근접하며 단순 oriented-kNN보다 나아야 함 |
| MC2: selective complementary dynamics | Global rank를 무작정 늘리지 않고 필요한 Local만 실행한다 | 같은 p95 dynamics latency에서 strong Global보다 유용한 국소 움직임이나 더 좋은 quality–latency 영역을 만들어야 함 |
| SYS: direct Gaussian transport | 적은 anchor의 움직임을 많은 Gaussian으로 직접 전달한다 | Gaussian 수와 dynamics cost의 분리, transport/render 비용의 분리 측정과 scaling evidence |

Structural backend, quasi-steady aero, selector score와 affine transport 각각의 독립적인 최초성을 주장하지
않는다. 방어할 아이디어는 이들을 static GS용 sparse scaffold와 selective complementary execution으로
결합하고, 실제 Pareto로 검증하는 데 있다.

## 13. Gate A/B/C가 묻는 질문

| Gate | 핵심 질문 | 실패했을 때의 축소 경로 |
|---|---|---|
| Gate A — Representation | 예측한 scaffold가 물리 표현으로 반복 사용 가능한가? | MC1을 줄이거나 철회하고 평가용 source/Oracle scaffold에서 MC2를 진단하며, 배포 경로는 explicit scaffold fallback으로 제한 |
| Gate B — Local necessity | Local 표현 자체가 strong Global보다 정말 필요한가? | MC2와 selector 개발을 중단하고 Global-only 방향으로 축소 |
| Gate C — Deployable gain | 실제 deterministic selector까지 포함해 same-anchor baseline보다 품질–속도 이득이 있는가? | Deployable/speed contribution을 주장하지 않음 |

Gate B의 privileged teacher는 배포 방법이 아니다. Local이 존재할 가치가 있는지를 먼저 확인하는
scope-control 진단이다. Gate C가 실제 저렴한 selector의 유효성을 판단한다.

Gate A는 사전 등록한 package attempt 중 실제 accepted 비율도 함께 본다. Gate C는 사전 등록한
sequence attempt 중 끝까지 성공한 비율과 end-to-end Pareto를 함께 본다. Operating envelope 밖의 입력은
결과를 보기 전 pre-check에서만 OOD로 분류한다. 범위 안에서 발생한 package rejection, 발산, surface나
transport 실패는 denominator에서 빼지 않으며, 성공 run의 좋은 conditional 품질로 높은 실패율을
상쇄할 수 없다. SYS scaling evidence만으로 MC2 또는 Gate C를 통과한 것으로 보지도 않는다.

S0–S7은 Gate A/B/C와 목적이 다르다. 단위·저장, scaffold, 구조 안정성, basis conditioning, aero,
state continuity, transport와 timing 측정이 깨지지 않았는지 확인하는 구현 안전검사다. S0–S7을
통과했다고 연구 기여가 입증되는 것은 아니다.

## 14. 속도를 위해 의도적으로 포기한 것

다음은 누락이라기보다 Minimal V0의 범위 선택이다.

- 정량 core는 flat-rest, single-layer, hard-attached strip과 rectangular flag다.
- Gravity는 core에서 끄고 zero-wind rest recovery를 명확히 본다.
- 구조는 한 번의 fixed-Hessian PD-style stretch/bending step이다.
- 바람은 prescribed one-way quasi-steady model이며 wake와 유체 이력을 풀지 않는다.
- 새 geometry는 다음 frame 바람에 반영하는 one-frame stagger를 쓴다.
- Selector는 미래 error나 전체 coupling을 보지 않는 저렴한 현재-load proxy다.
- Global/Local basis는 Setup 뒤 고정하며 frame마다 다시 만들지 않는다.
- Frame당 coupled solve는 한 번뿐이다.
- Gaussian transport는 얇은 표면에서 식별 가능한 제한된 affine 변형만 사용한다.
- 정량 appearance는 degree-0 color만 사용한다.

다음도 명시적으로 주장하지 않는다.

- Engineering-grade stress, force, torque, support reaction 또는 material identification
- Full nonlinear shell, anisotropy와 pre-curved shell의 정량 정확도
- Wake, vortex shedding, 정확한 lift 또는 two-way FSI
- Self-contact, tearing, plastic crease와 detached flight
- Exact energy conservation, exact modal recovery와 certified error adaptivity
- Fully mesh-free physics 또는 모든 얇은 물체로의 일반화
- 실측 end-to-end p95 전의 real-time 주장

## 15. 지금 고정된 것과 아직 증명되지 않은 것

### 현재 설계에서 고정된 것

- Setup/runtime의 다섯 단계 구조
- Flat-rest core 범위와 입력 contract
- Stretch/bending, 질량, damping의 최소 backend
- Global basis와 complementary Local basis의 역할
- Full-anchor wind와 single coupled solve
- Deterministic selector 및 Active/Decay state policy
- Anchor-to-Gaussian transport와 안전/failure 기록
- Gate A/B/C 및 실패 시 claim 축소 경로

### 실험으로 아직 답해야 하는 것

- Independent GS마다 predicted scaffold가 안정적으로 만들어지는가?
- 단순 oriented-kNN이나 explicit mesh extraction보다 MC1이 실제로 유리한가?
- Local이 strong Global rank보다 같은 p95 dynamics latency에서 의미 있는 detail을 복원하는가?
- Cheap deterministic selector와 privileged teacher 사이의 차이가 충분히 작은가?
- Decay가 popping을 줄이면서 solve rank와 p95를 지나치게 키우지 않는가?
- One-frame stagger가 눈에 보이는 phase lag를 만들지 않는가?
- Full-anchor aero와 Gaussian transport 비용을 포함한 end-to-end 이득이 남는가?
- Degree-0 appearance로도 정량 core 영상이 충분히 설득력 있는가?

## 16. Deferred 항목과 다시 여는 조건

| Deferred 항목 | 다시 검토하는 조건 |
|---|---|
| Aero hyper-reduction H | Full-anchor aero가 synchronized p95 dynamics의 실제 병목일 때 |
| Missing-force network F | Physics-only Local에 반복 가능하고 눈에 띄는 결함이 남을 때 |
| Learned selector G | Privileged teacher와 deterministic selector의 동일 비용 차이가 클 때 |
| KKT/Schur | F가 겹치는 nodal patch force를 실제 출력하도록 채택될 때 |
| Same-frame corrector | One-frame lag가 보이는 phase jump나 불안정을 만들고 same-p95에서 개선될 때 |
| Confidence calibrator | Hard reject/fallback rate가 MC1을 실제로 막을 때 |
| Teacher-FSI | Wake 또는 unsteady-aero claim을 추가하기로 결정할 때 |
| Advanced solver | 작은 direct solve의 factorization이 profiler에서 실제 병목일 때 |

이론적 edge case만으로는 deferred 항목을 승격하지 않는다. 눈에 보이는 실패 제거, MC1/MC2를 막는
문제 해결 또는 측정된 quality–latency 개선이 있어야 한다.

Gate evidence 전에는 명백한 수식 오류 정정 외에 새 method equation, backend 또는 runtime stage를
추가하지 않는다. 새 core stage가 정말 필요하다면 기존 다섯 단계 중 하나를 대체해야 하며 병렬
mainline을 만들지 않는다.

## 17. 내가 이 아이디어를 검토할 때 답해야 할 질문

### 핵심 연구 가설

1. Target mesh 없이 static GS에서 sparse physical scaffold를 만드는 문제가 충분히 가치 있는가?
2. MC1이 실패해 source/Oracle scaffold만 남아도 MC2 연구는 여전히 가치 있는가?
3. Local이 Global rank를 늘리는 것보다 유리할 것이라는 직관이 명확한가?
4. Local이 필요하더라도 cheap selector가 그 이득을 유지할 가능성이 있는가?
5. MC1과 MC2 중 하나만 통과했을 때도 논문으로 축소 가능한가?

### 시각적 타당성

6. Full-anchor quasi-steady wind만으로 silhouette, tip trajectory, flutter와 temporal coherence가 충분한가?
7. One-frame lag가 영상에서 실제로 보이는가, 아니면 이론적 우려에 그치는가?
8. Active/Decay 전환에서 pop 또는 activation delay가 거슬리는가?
9. Anchor-to-Gaussian transport가 declared normal extension 범위에서 silhouette와 covariance를
   안정적으로 유지하는가? Arbitrary thickness deformation은 평가 대상으로 오해하고 있지 않은가?
10. Degree-0 appearance가 core 영상을 평가하기에 충분한가?

### 성능과 공정성

11. Strong Global rank sweep과 같은 p95 dynamics latency에서 비교하고 있는가?
12. Same-anchor full solve, dynamics-only, transport와 render 시간을 각각 측정하는가?
13. Fixed Active 수와 별개로 Decay를 포함한 actual solve rank를 기록하는가?
14. Full-anchor aero 또는 Gaussian transport가 병목이 되어 Local 절약을 상쇄하지 않는가?
15. 성공 run만 남기지 않고 package rejection과 sequence failure를 denominator에 유지하는가?

### Scope control

16. 새 지적이 MC1/MC2를 실제로 무효화하는가, 아니면 더 높은 물리 정확도를 요구할 뿐인가?
17. 새 모듈 없이 baseline, limitation 또는 failure report로 답할 수 있는가?
18. Deferred 항목의 재검토 조건이 실제 측정으로 충족되었는가?
19. 새 runtime stage를 넣는다면 기존 다섯 단계 중 무엇을 대체할 것인가?
20. Gate A/B/C 결과가 나오기 전에 정말 해결해야 하는 문제인가?

## 18. 최종적으로 기억할 세 문장

1. 이 방법은 많은 Gaussian을 직접 시뮬레이션하지 않고, sparse anchor에서 움직임을 계산해 Gaussian으로 전달한다.
2. 바람은 모든 anchor에 계산하지만, Global이 놓치는 국소 움직임은 현재 필요한 Local mode만 선택해 한 번의 coupled solve에서 보충한다.
3. 성공 여부는 물리의 완전성이 아니라 scaffold의 반복성, same-p95 dynamics에서의 Local 필요성,
   scaling evidence와 final deployable quality–latency 측정으로 판단한다.

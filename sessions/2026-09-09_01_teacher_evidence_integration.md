# 2026-09-09 01 Teacher 전체 개발 근거의 canonical 문서 통합

## 현재 상태

확인 기준: 2026-09-11. 사용자 요청으로 스케치·master·R0–R7을 최근 결정·구현·저장 결과와 대조해 누락과 오래된 상태 표시를 보정했다.
- 현행 방법·claim은 [ideas index](../README.md), R1 계약·채택 경계는 [R1](../development/r1_teacher_probe_oracle.tex)를 따른다.
- 사용자30fps 목표, Teacher 비용과 online 비용의 구분, CPU 풀이+HVP/graph 채택과 전체 성능 비교를 반영했다.
- 4배 보완의 공간 미달과10초 Δt 추천 없음은 저장 결과의 확인 범위까지 명시했다. 전체 물리 재검산·원인 확정·R1 채택 근거로 승격하지 않았다.
- R1/R2에서 조기 성능을 확인하자는 의견은 미채택 제안으로 남겼다. 물리식·검증 허용오차·단계 순서와 완료 체크는 바꾸지 않았다.
- 다음은 [code 인수인계](../../code/sessions/2026-09-10_02_teacher_timestep_search.md)의 결과 검토·원인 분석이다. 아래 과거 단계의 다음 작업을 현행 지시로 사용하지 않는다.

## 2026-09-11 문서 감사 범위

| 문서 | 누락/불일치와 반영 | 유지한 경계 |
|---|---|---|
| 스케치 | 최소30fps, setup/매 frame 비용, 후속4배 결과와10초 탐색을 연결 | 실시간 달성 미입증; 허용 범위의 바람 조작 우선 |
| R1 | 이미 구현된 응력·요소 연결·solver를 미구현으로 표시한 요약 보정; HVP/graph와 CuPy 미채택, 전체 성능 비교, segmented producer 보존,10초 탐색 계약/결과 추가 | 수식 그대로; 공간 미달·시간 한도와 수치 실패 구분; 독립 기준·고차 map/oracle 미완료 |
| R2 | 사용자10–20초 목표와 accepted horizon 미동결 구분; 조기 비용 확인 의견 기록 | 제안 미채택, R1 전 학습 불가, rank/성능 설정 미정 |
| R7 | 전체 frame 약33.3ms 목표, 분포·초과 비율·입력 지연과 측정 조건 명시 | Teacher 비용·evaluator 단독 속도로30fps를 주장하지 않음 |
| Master/공통 표시 | R1 현재 상태와30fps 목표 소유권 연결; 문서별 체크리스트 날짜 지원 | R0/R1 진행, R2–R7 대기; 완료 flag 유지 |
| R0/R3–R6 | 관련 입력·동결 계약·완료 기준 대조에서 이번 작업으로 변경할 계약 없음 | source 본문 유지; shared preamble 의존 PDF는 재빌드 |

실험 상세의 소유 문서는 [전체 성능 비교](../../experiments/R1_teacher_velocity_reset/p3_shell_random/profiling/full_run/README.md),
[4배 보완](../../experiments/R1_teacher_velocity_reset/p3_shell_random/scale4/fast_handoff/README.md),
[10초 탐색](../../experiments/R1_teacher_velocity_reset/timestep_search/README.md)다.
후속 두 실행은 인계 시 보존한 `evidence/handoff_20260911`의 metadata 확인이며 전체 NPZ/물리 재검산을 이번에 수행하지 않았다.
재현 명령·source hash와 상세 수치는 해당 실험 문서에 두고, ideas에는 채택 판단에 필요한 근거와 경계만 연결했다.

이번 변경은 기존 사용자 문서 작업을 이어받은 감사다. 별도 아이디어 초안·archive·실험 원본·구현은 수정하지 않았으며 commit/push/fetch도 하지 않았다.
검증 완료: owning source10개를 `latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error`로 실제 빌드했다.
독립 빌드는 최대3개를 병렬 실행했다. Sketch30쪽, master7쪽, R1 32쪽, R2 11쪽, R7 8쪽이며 모든 PDF가 non-empty다.
최종 log에 미해결 참조·중복 label·missing character·overfull·LaTeX error가 없었다.
Master/R0–R7의2쪽 체크리스트와0/8 합계를 확인하고 R1의 실제 페이지 배치를 시각적으로 점검했다.
스케치/R0–R7의 기존 display equation과 완료 flag를 변경 전 snapshot과 대조했다. R0/R3–R6 source 본문은 그대로다.
Sketch/master bundle3개/20개 항목의 CRC와 현재 source/PDF byte 일치를 확인했고 Markdown 로컬 링크45개와 개인 절대 경로 부재를 검사했다.
Ideas의 `git diff --check`를 확인했으며 stage/commit/push는 하지 않았다. 구현 테스트·시뮬레이션 재실행은 이번 문서 감사 범위 밖이다.

## 이전 단계의 결정과 근거

## 요청과 인수 범위

Wind3DGS ideas-side. 사용자는 이 채팅의 작업 전체를 인수인계뿐 아니라 아이디어 스케치에도 자세히 반영해
나중에 논문 작성 시 참고할 수 있도록 요청했다. Registry부터 GPU, 구조·시간·공간 진단, sample과 P3 보완을 정리하고
스케치의 관련 계약을 직접 보정했다. 처음 만들었던 별도 구현 기록은 후속 요청에 따라
[R1 통합 TeX](../development/r1_teacher_probe_oracle.tex) /
[PDF](../development/r1_teacher_probe_oracle.pdf)의 해당 본문 절에 흡수했다. 현재 전달 문서는 R1 하나다.

작업 시작 시 ideas HEAD는 `a9c11770d2746964d16b6d9454ca57e7c6970297`이었다.
현재 README가 가리키는 스케치의 선행 리뷰 수정, master/R0–R7 분할 문서와 shared preamble이
이미 로컬 초안으로 존재했다. 사용자가 지정한 현재 canonical 문서의 선행 상태로 인수하고 기존 내용을 보존했다.
이번 commit에는 독립 빌드 가능한 canonical 문서 집합과 그 의존성을 함께 보존한다.
R0/R2–R7 source/PDF와 shared preamble의 본문은 수정하거나 새로 실행하지 않았다.
선행 수정 자체를 이번 채팅의 Teacher 실험 결과나 완료 근거로 주장하지 않는다.
별도 `idea_sketch_2_solver_free_response_operator.tex`, `rdgl_review.tex` 및 기존 08-22 session 변경은 제외한다.
새 method pivot이나 archive 삭제가 아니며 target-mesh-free learned response 방향은 유지한다.

## 반영한 판단과 근거

| 항목 | 판정 | 근거·문서 영향 | 미완료/재검토 조건 |
| --- | --- | --- | --- |
| Teacher vertex=sample 설명 | 보정 수용 | 고차 DOF와 positive quadrature가 다름. 스케치에 consistent mass 식과 total mass/양정성 검사를 명시 | 최종 고차 registry/validator 구현 |
| 모든 map의 nonnegative 제약 | 조건부 보정 | GS/기존 P1 convex map은 유지. 고차 Teacher shape만 signed weight, 별도 law/version·reproduction·conditioning·adjoint 허용 | P3 public 10-shape map/trajectory 연결 |
| 정적 에너지 통과의 해석 | 제한 명시 | Native/area hinge 반례, quadratic patch interior weak-force 결함을 기록 | 내부 힘/방향/동적 velocity 검사로 재판정 |
| 시간+공간 acceptance | 계약 명확화 | 두 축은 동일 structure/element/mass/BC/load identity여야 함 | 기존 shell 시간 통과를 P3 공간 통과와 합치지 않음 |
| P3 결과 | 제한된 개발 근거 수용 | Prescribed pressure의 공간/방향/spline 기준과 연속 시간 상계 1% 통과 | Nonlinear membrane+bending와 실제 held-wind 미연결 |
| 원래 x² 초기 상태 | 실패 보존 | P3/spline에서도 속도 수렴 실패, broad-band energy 확인 | 유효 초기 상태를 삭제·필터링하거나 tolerance 완화하지 않음 |
| 학습 sample | 개발 경로 검증 완료 | 세 Newton source/15 window, source/replay/loader 확인 | `training_eligible=false`, R2 진입 불가 |
| 첫 accepted 입력군 | 최종 범위 보류, 다음 탐색 선택 | Rest-start 변화 바람에서 도달한 형상을 velocity-reset하여 비교하기로 결정 | 탐색 실행·수렴·학습 이득 미검증; 기존 displaced decay 요구 유지 |
| R1 상태 | 현행화 | “fixture 미구현”에서 개발 진단 진행으로 변경, canonical acceptance 체크박스 유지 | R0 잔여/동일 backend 수렴/GS oracle·transport·spectrum 필요 |

상세 연구 기록은 모든 단계의 수식, SI 단위, slope-free 경계, 초기 상태와 forcing family,
metric 분모와 sampled/continuous-time 차이, 성공·실패 수치, 원본 보존·hash·명령 및 논문 claim 경계를 포함한다.
Code의 09-06 01부터 09-08 07까지 전체 기능별 session과 experiment evidence 13개 폴더를 연결했다.
기존 기본 GUI 이후의 작업이 정량 reference/데이터 경로였다는 점과 새 P3 GUI가 없다는 점도 설명한다.

## 관련 구현·실험 보존점

- Code: `894993c5d44533938fb1f360acd27da880d5fb71` — 누적 Teacher 구현·14개 test module·설계 기록, 63개 파일.
- Experiments: `c7f4c74d434a2648b54ffbd8bda475e8efdbd6a3` — GPU 이후 12개 실험 폴더와 compact evidence/session, 156개 파일.
- 실행 당시 code/experiments HEAD는 기존 `7010682`/`61d972e`였으며 실제 dirty executable snapshot은
  각 environment의 content hash로 고정돼 있다. 후속 commit ID로 frozen run provenance를 덮어쓰지 않았다.
- [Code checkpoint](../../code/sessions/2026-09-09_01_teacher_checkpoint.md),
  [experiment checkpoint](../../experiments/sessions/2026-09-09_01_teacher_checkpoint.md)를 연결한다.

## 최초 분리본의 문서·PDF·bundle 검증

이 절은 ideas commit `00edc1c`의 최초 분리본 제작 당시 기록이다. 아래 별도 기록의 빌드 명령과
4개 PDF/22개 bundle entry는 그 시점에 한정되며, 현행 통합본의 빌드·검증은 후속 절을 따른다.

이번에 수정한 canonical sketch, master roadmap, R1 명세 및 새 연구 기록 네 문서를 실제 XeLaTeX로 빌드했다.
각 명령은 ideas root에서 실행하며 `latexmk`의 필요한 bibliography/reference pass까지 성공해야 한다.

```bash
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  development/r1_teacher_probe_oracle.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  development/r1_teacher_implementation_record.tex
```

긴 hash/path와 표의 줄바꿈을 보정했고 연구 기록의 결과 표·수식·hash/명령 페이지를 raster preview로 확인했다.
최종 로그에서 undefined reference/citation, overfull과 missing glyph가 없음을 확인했다.
최종 PDF는 sketch 27쪽, master 6쪽, R1 명세 9쪽, 동반 연구 기록 11쪽이며 모두 non-empty다.
Sketch bundle에는 TeX/PDF/Bib 3개, master bundle에는 master TeX/PDF, development README/shared preamble,
R0–R7 TeX/PDF 16개와 동반 연구 기록 TeX/PDF 2개, 총 22개를 넣고 각 entry byte를 실제 파일과 대조했다.
그 밖의 임시 build output이나 별도 검토 초안은 bundle에 넣지 않는다.

누적 코드 **234개 검사 / 95.133초 / OK**, sample 원본 대조·batch 재로드,
source hash 271개 항목 및 원본 9개 run의 7,777 inventory/5.28 GB hash 검산은 code/experiment checkpoint에 기록했다.
이번 문서 작업에서 구현 source와 frozen experiment evidence를 수정하지 않았다.
Root 및 무관한 dirty 파일을 보존했고, code/experiments의 session index는 관련 항목만 부분 stage했다.
외부 자료는 새로 fetch/다운로드하지 않고 기존 로컬 source·보고서를 사용했다.
새 remote push/fetch는 수행하지 않으며 현재 원격 최신성을 주장하지 않는다.

## 남은 작업

다음 작업은 아래 후속 결정의 rest-start 변화 바람/velocity-reset 비교다. 첫 실행에서 사용할 backend의
wind/restart 지원과 개발/accepted 상태를 확인한다. P3 nonlinear wind/consistent mass/고차 map 및 동일 backend
수렴은 여전히 남은 선행 조건이며, 개발용 탐색으로 시작하면 그 결과를 accepted 학습데이터로 승계하지 않는다.
이 문서 통합으로 학습 적격성을 바꾸지 않는다. 다음 판정도 같은 R1 문서와 evidence 계보에 누적한다.

## 후속 요청: 구현 기록을 R1의 해당 본문에 흡수

사용자는 R1 문서가 둘인 이유를 확인한 뒤, 구현 기록을 기존 R1에 흡수하고 달라진 수식을 해당 위치에서
직접 교체하며 필요에 따라 코멘트를 붙이도록 요청했다. 단일 실행 문서의 큰 절 8개는 유지했다.
별도 문서를 통째로 뒤에 붙이는 대신 아래처럼 내용을 소유 절로 옮겼다.

| 이전 구현 기록의 내용 | 통합 R1 위치 |
| --- | --- |
| 연구 질문·작업 범위 | 1절 목적과 소유권 |
| Teacher mass/positive quadrature·signed shape map | 3절 현행 계약과 수식 |
| Registry·trajectory·sequence·초기 변위 | 4.2 TeacherPhysicsRegistry와 reference 생성 |
| Common probe·비교기 구현 | 4.3 Canonical probe와 mapping |
| 개발 sample 15개 생성·검증 | 4.7 개발용 sample과 학습 적격성 |
| P3/spline 구조식·quadrature·경계 | 5.1 현행 구조 검증 후보 |
| Acceleration Newmark 갱신식·precision 보정 이유 | 5.2 현행 CPU 시간 적분 |
| Native/area hinge·quadratic patch·nonlinear shell·interior weak-force 반례 | 5.3 구조 후보의 교체 이유와 이전 식의 적용 범위 |
| 구현 계보·원본·command·hash·전체 evidence index | 6절 산출물 |
| GPU·시간·공간·두 입력군/연속 시간 상계 결과 | 7절 필수 fixture와 metric |
| 채택 경계·입력 범위 미결정·후속 구현 | 8절 종료 조건과 실패 경로 |

- 수식은 한 곳에서 정의해 참조한다. Quadrature mass `m_q`, full consistent mass와 constrained positivity를
  명시했고, map의 sign 조건은 representation/law별로 분리했다.
- 현행 P3 C0IP 식과 acceleration-form Newmark 식을 해당 설계 절에 직접 배치했다.
  같은 KL constitutive 식과 forward/adjoint 식의 중복 정의는 참조로 정리했다.
- 바뀐 위치에 `[수식 변경 이유]` TeX 주석과 PDF에서 읽는 적용 범위/설명을 붙였다.
  이전 patch/hinge 식은 실패·교체 근거 절에만 당시 scope로 남긴다.
- P3가 최종 nonlinear wind Teacher로 채택된 것은 아니다. 기존 시간 통과와 새 공간 통과를 합치지 않고,
  원래 x² 속도 실패, development eligibility false 및 R1 종료 체크박스를 유지했다.
- 별도 `r1_teacher_implementation_record.tex`/PDF는 통합 확인 후 제거했다.
  제거 전 HEAD byte와 일치함을 확인했으며 원본은 `00edc1c`에서 복구 가능하다.
- Sketch/master/index, PDF allowlist와 두 bundle을 단일 R1로 맞추고 code/experiments의 README·checkpoint 링크를 갱신했다.
  동일 논리 작업이므로 새 session note를 추가하지 않고 각 기존 note에 후속 내용을 붙였다.

통합 전 기록의 14개 항목을 모두 배치했다. 원래 숫자 토큰 239종, 코드/path literal 48종과 대표 SHA-256 5개가
통합 source에 남아 있는지 대조했다. 원래 R1의 acceptance 항목과 기존 수학적 계약도 유지한다.
수치 토큰 검사는 내용 검토의 보조 검사이며 물리 검증을 다시 수행한 것으로 해석하지 않는다.

현행 빌드는 ideas root에서 다음 세 명령을 사용한다.

```bash
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  development/r1_teacher_probe_oracle.tex
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex
```

최종 R1 PDF는 19쪽, sketch는 27쪽, master는 6쪽으로 모두 실제 빌드에 성공했다.
최종 로그의 undefined reference/citation, missing glyph와 overfull은 없었다.
현행 P3/시간 갱신식과 채택 조건 페이지의 raster preview도 확인했다.
Sketch bundle 3개 entry와 master bundle 20개 entry를 source/PDF의 byte와 대조했고,
두 bundle에 제거한 구현 기록이 남지 않았음을 확인했다.
R1 TeX/PDF 한 쌍만 남았고, 기존 큰 절 8개 및 acceptance checkbox 38개를 유지했다.
변경 문서의 로컬 링크 123개, source 내 교차 참조와 whitespace 검사가 통과했다.
이번 문서 변경에 맞춰 코드 테스트나 물리 실험을 새로 실행하지는 않았다.
R0/R2–R7, shared preamble, 구현 source/tests와 frozen experiment artifact는 변경하지 않았다.
무관한 기존 dirty 파일과 root는 보존한다. 이번에도 fetch/push와 새 물리 실험은 수행하지 않는다.

## 후속 결정: rest-start 변화 바람과 중간 형상의 velocity-reset 비교

### 대화에서 구분한 질문과 사용자 결정

사용자는 최종 사용 상황에서 이미 변형된 상태도 새로운 외력에 대응해야 하므로, 바람 방향을 주기적으로
랜덤화하고 시간·패치별 데이터를 생성해야 하지 않는지 질문했다. 이어 rest-start 랜덤 바람 데이터와
변형된 상태에서 시작한 동일한 바람 데이터가 학습 후 어떤 차이를 만드는지 확인했다.
마지막으로 오늘의 구현·실험은 종료하고, 다음 작업을 다음 방향으로 진행하기로 했다.

> Rest position에서 시뮬레이션을 진행하고, 중간중간 변형된 position에서 속도를 0으로 만든 뒤
> 시뮬레이션을 계속하여 적절성 여부를 판단한다.

선택한 것은 **다음 탐색의 순서**다. 아직 이 비교를 실행하지 않았고, 최종 학습 데이터 구성·혼합 비율,
모든 초기 변형의 허용 또는 임의 형상에서 시작하는 target runtime 기능을 확정한 것은 아니다.
이번 후속 작업은 ideas 문서 기록만 수행하며 code 구현/실험 run을 시작하지 않는다.

| 항목 | 판정 | 근거·문서 영향 | 검증 또는 재검토 조건 |
| --- | --- | --- | --- |
| 변형·운동 중 새 바람에 대응 | 목적 수용 | 정지 시작은 데이터의 시작 조건일 뿐 전체 궤적이 정지 상태인 것은 아님. Sketch의 초기 상태 분포 절에 명시 | Supported envelope의 다양한 wind와 장기 rollout 검증 |
| Rest-start와 변형 시작의 차이 | 조건부 해석 | 같은 rest/재료/고정부·완전한 상태·이후 바람이면 동일 동역학. 형상만 같고 속도가 다르면 다른 상태이며, 차이는 주로 상태 분포와 관측 빈도 | Rest-start에서 드문 상태의 보충 효과를 같은 학습 예산으로 비교 |
| 중간 형상에서 속도 0 | 다음 탐색 수용 | 원래 rest 기준과 현재 위치를 보존하는 개입. R1에 분기 절차와 kinetic-energy 제거 식을 추가 | 재시작 일관성, 위치·고정부, mass/에너지와 시간·공간 수렴 |
| 랜덤 바람의 구체화 | 후보 유지 | 방향 외 세기·변화 간격·보간·seed도 기록. 부드러운 전환/급반전을 구분하고 국소 모드가 부족하면 traveling/local gust 검토 | 입력 대역·세기·시각·horizon은 다음 실행 전에 명시; 수치 값 미동결 |
| 시간·패치 추출 | 전체 궤적 기반 원칙 수용 | 주변 당김·고정부·Global 영향을 유지하고 전체 물체에서 관측을 추출. Local은 frozen Global의 residual이라는 기존 소유권 유지 | Reset 경계를 일반 연속 label로 섞지 않으며 동일 원본의 파생 분기 split 누출 방지 |
| 초기 상태 데이터의 추가 이득 | 보류 | 다양한 초기 상태가 항상 정확도를 높이지 않으며 현재 응답 package의 표현 한계도 가능 | Converged Teacher의 network-free oracle, student 초기화 계약, 이후 rest-only/혼합 비교 |
| 기존 R1 실패·Gate | 유지 | Prescribed-pressure 통과는 실제 wind 또는 x² decay 통과가 아님. 새 탐색도 수렴 실패를 삭제하는 근거가 아님 | 기존 x² 실패와 aero-off displaced decay 요구 유지; `training_eligible=false` 유지 |

### 다음 실행의 최소 비교 절차

1. 다음 담당자는 이 후속 결정과 R1의 `sec:r1-velocity-reset-plan`을 먼저 읽는다. 사용할 backend의
   실제 wind 지원·checkpoint/restart 경로와 현재 개발/accepted 상태를 확인한다.
2. 원래 rest geometry/material/attachment에서 전체 물체의 변화 바람 궤적을 생성한다.
   물리 시각 `t_j`를 지정해 완전한 상태를 저장하고, 개입 없이 restart한 경우 원본과 일치하는지 먼저 검사한다.
3. 각 시점에서 자연 연속과 전체 자유 DOF velocity-zero 분기를 비교한다. 원본의 여러 checkpoint에서
   독립적으로 분기하는 것을 기본 대조로 두고, 한 분기에 여러 번 reset하면 누적 개입으로 별도 식별한다.
4. 바람의 절대 시각·seed는 유지한다. 속도 reset 뒤 frame-start traction을 평가하므로 두 분기의 실제 force는
   다를 수 있다. Backend에 따라 acceleration·적분 이력·force cache의 일관된 재구성이 필요하다.
5. 개입 순간 position/rest/pin은 유지되고 kinetic energy는 `0.5 vᵀ M v`만큼 제거되는지 확인한다.
   이 개입은 자연 감쇠, zero ambient, aero-off 또는 일반 시간 적분 오차와 구분하여 기록한다.
6. 후속 displacement/velocity·위상·회복·guard/finite와 동일 backend의 refinement를 비교한다.
   Reset은 다른 초기 운동 상태를 만드는 것이므로 자연 연속 분기와의 궤적 일치가 합격 기준은 아니다.
7. 물리/수치 적절성을 확인한 뒤 patch/window 생성과 student/oracle 재시작을 설계한다.
   Reset을 가로지르는 불연속을 일반 label로 숨기지 않고 같은 원본과 파생 분기는 같은 source group에 둔다.

이번 설명의 외부 확인은 앞선 읽기 전용 답변에서 실제 웹으로 조회한
[GNS 논문](https://proceedings.mlr.press/v119/sanchez-gonzalez20a.html)과
[MeshGraphNets cloth 공식 구현](https://github.com/google-deepmind/deepmind-research/blob/master/meshgraphnets/cloth_model.py)을 참고했다.
전자는 장기 rollout 오차, 후자는 현재/기준 형상과 속도 정보의 구분을 확인하는 보조 근거다.
두 모델의 next-state 학습 방식을 현재 setup-once response package로 도입하기로 결정한 것은 아니다.
이번 기록 단계에서는 외부 자료나 Git 원격을 새로 fetch/다운로드하지 않았다.

### 문서 반영과 검증

- Canonical sketch: 기존 accepted input 범위의 미동결 상태를 유지하고 사용자 선택 및 데이터/상태 해석을 해당 Teacher 장에 추가.
- R1: 입력군 선택 표와 다음 진입 조건을 갱신하고 분기·재시작·개입 에너지·데이터 판정 절차를 해당 설계 절에 추가.
- Ideas README와 기존 session/index를 갱신. Stage dependency/Gate/claim은 바뀌지 않아 master source 및 R0/R2–R7은 유지.
- 수정한 sketch/R1 PDF를 실제 빌드하고 두 canonical bundle의 해당 파일을 동기화했다.
- 오늘 새 시뮬레이션·학습은 수행하지 않으며 이번 후속 문서 수정의 commit/push도 수행하지 않는다.

이 후속 수정의 최종 PDF는 sketch 28쪽, R1 20쪽이다. 아래 명령은 모두 성공 종료했으며
non-empty PDF와 새 절의 텍스트를 확인했다. 최종 pass의 undefined reference/citation,
missing glyph 및 overfull은 없다.

```bash
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  development/r1_teacher_probe_oracle.tex
latexmk -cd -xelatex -interaction=nonstopmode -halt-on-error \
  development/r1_teacher_probe_oracle.tex
```

마지막 명령은 R1 앞부분의 미결정 설명을 이번 선택과 일치시킨 후 재빌드한 것이다.
Sketch bundle 3개 entry, master bundle 20개 entry는 모두 현재 source/PDF와 byte가 일치한다.
Master source/PDF는 그대로 유지하고 master bundle 안의 R1 source/PDF만 동기화했다.
Git whitespace와 문서 로컬 링크를 검사하고 작업 전부터 있던 dirty/untracked 파일의 hash를 보존했다.
Root/code/experiments에는 이번 후속 작업으로 생긴 변경이 없다. Ideas의 이번 문서·산출물 변경은 미커밋 상태다.

## 후속 요청: 전체 문서 재감사와 표지 다음 개발 체크리스트

사용자는 이 채팅의 작업이 아이디어 스케치와 개발 문서에 잘 반영되었는지 다시 확인하고 누락을 보완하며,
개발 문서의 표지 다음에 장별 완료 체크 페이지를 추가하도록 요청했다. 앞선 velocity-reset 계획 수정도
이번 작업의 선행 상태로 인수했다. 연구 방법의 pivot이나 새 실험 실행은 아니다.

### 재감사 범위와 보완

Root와 읽기 전용 보조 검토 두 개로 역할을 나누었다. 한 검토는 code/experiments의 09-06~09-09
구현·실험 기록과 필요한 실제 source를 R1에 대조했고, 다른 검토는 sketch/master/R0/R2–R7의 소비 계약과
장 전체 완료 근거를 감사했다. 수정은 ideas에 한정했다. 주요 계보의 coverage는 다음과 같다.

| 이 채팅의 작업 | 현재 상세 근거 소유 위치 | 감사 결과 |
| --- | --- | --- |
| Registry, native SI/hash/model validator, optional import | R1 Registry/reference 생성 절 | 기존 계보 유지; native 법칙·단위·환산 미구현 경계 보완 |
| Trajectory T/T+1, 실패 prefix/replay, wind program, 초기 변위 | R1 trajectory/sequence 절 | 기존 계보 유지; setup-fixed direction과 seed metadata 제한 보완 |
| P1 common probe, forward/adjoint와 비교기 | R1 공통 계약 및 mapping 구현 절 | 이미 반영됨; 고차 Teacher/GS 소비 경계를 R0/R3에 연결 |
| 사용자 GPU 12/12 | R1 필수 fixture/GPU 절 | 이미 반영됨; 실행 가능성과 물리 수렴 구분 유지 |
| Native/area hinge, quadratic patch, 내부 힘 반례 | R1 구조 후보 교체 절 | 수식·실패 원인·대체 후보 계보가 반영됨 |
| DOP853, cancellation과 acceleration Newmark, 시간 refinement | R1 현행 적분식 및 시간 결과 절 | 이미 반영됨; step 5의 직접 잔차 수치 근거 보강 |
| 선형 exact-modal 공간/방향 실패 | R1 공간 응답 실패 절 | 결과·분모·실패 범위가 반영됨 |
| 3개 source/15개 개발 window 및 loader/replay | R1 sample 절 | 이미 반영됨; R2 진입 문서에도 eligibility 차단 명시 |
| P2/P3/spline, consistent mass와 signed shape, 연속 시간 상계 | R1 질량/map 계약, 현행 판 및 공간 norm 절 | 초기 변위/속도 표현과 교차항 적분 해석 보정 |
| Rest-start와 변형 상태의 차이, velocity-reset 다음 탐색 | Sketch 초기 상태 절 및 R1 velocity-reset 절 | 선택·미검증 경계 유지; downstream 초기화·lineage·residual 연결 |
| Raw/hash/명령과 누적 QA | R1 산출물/provenance 절 | 234검사·원본 inventory 검산을 상세 checkpoint에서 본문으로 보강 |

| 누락 또는 오독 가능성 | 반영한 보정 | 검증·비주장 경계 |
| --- | --- | --- |
| Sketch 총질량 합 식의 DOF 범위 | `1ᵀ M 1 = M_ref`는 고정부 제거 전 전체 행렬임을 명시 | 허용 DOF의 양정성과 총합을 구분; student diagonal mass는 유지 |
| Native material 식의 생략 | `newton_stable_neo_hookean_membrane_v1`, `tri_ke/tri_ka` N/m, `tri_kd/edge_kd` s, `edge_ke` N 기록 | Native tuple의 continuum E/ν/h/ρ 환산 미구현; CPU StVK/P3와 별개 |
| Seed가 방향을 랜덤화한다는 오독 | 현재 scalar speed program과 setup-fixed direction을 명시 | Vector wind/RNG 상태·program version은 다음 구현 |
| x² 실험을 초기 속도 입력으로 오독 | 초기 변위 `u_z^0=10^-3 x²`, `v^0=0`에서 생긴 속도 응답 수렴 실패로 교체 | 기존 실패 수치·원인을 삭제하지 않음 |
| P3–spline Duffy6 범위 | Degree≤9는 교차항, 각 self-norm은 자체 mass라고 명시 | 실제 평가 코드와 일치; 새 적분법 도입 아님 |
| Precision 보정의 직접 근거 부족 | 새 step 5 잔차 3.12274e-10, bound 1.34886e-8, 위치 역산 2.45592e-8 m/s², full N2560 속도 차이 3.31711e-9 추가 | 당시 실행 근거이며 이번 새 solver 실행 아님 |
| R0 전 R1 시작 금지 문구와 개발 진단의 충돌 | Canonical acceptance/학습 승격은 R0 이후, 격리된 개발 진단은 적격성 false로 구분 | R0 전체 완료나 accepted Teacher 생성 승인 아님 |
| Downstream의 입력 소비 경계 부족 | R0 표현형·초기화·lineage, R2 eligibility/시작 상태, R3 동일 Teacher identity, R4 split, R5 coupled residual, R6 reset 소유권 보완 | 새 초기화 알고리즘·데이터 혼합 비율·threshold 미동결 |
| R7의 “이미 통과한” 전제 | R7 전에 R1/R2에서 통과해야 할 전제로 교체 | 현재 base transport가 통과했다고 주장하지 않음 |

R1에 추가한 누적 QA 수치는 기존 checkpoint의 관련 14개 module/234검사/95.133초/OK,
source hash 271개·compact hash 60개·JSON 91개·Python 41개·shell 13개 및
원본 9개 run/7,777 inventory/5,283,975,129 byte 검산이다. 이번 감사에서는 그 기록과 문서를 대조했으며
전체 물리 실험이나 해당 코드 테스트를 새로 반복하지 않았다. 외부 웹 조회나 Git fetch/push도 수행하지 않았다.

### 체크리스트의 구조와 완료 판정

- Master와 R0–R7은 표지 다음인 **PDF 2쪽**에 각각 단계별/장별 체크리스트를 둔다. 그 뒤 목차와 본문이 이어진다.
- R0–R7은 기존 본문 8개 장과 일대일로 연결하고, 각 행에 완료 체크·본문 링크·상태·남은 조건을 표시한다.
  Master는 R0–R7 단계별 행을 두고 기존 본문 상태 표를 앞 페이지로 옮겨 중복 갱신을 피한다.
- 현재 canonical 장 전체를 완료 처리할 근거는 없으므로 각 0/8, master도 0/8이다.
  R0의 Teacher 하위 계약 및 R1 개발 진단은 진행으로 표시하고 R2–R7은 선행 조건 대기로 둔다.
  목적/명세가 문서에 존재한다는 사실을 구현 완료로 세지 않는다.
- R1은 별도 성과 box에 Registry/trajectory/sequence·P1 probe, GPU 12/12,
  15개 development sample 검증, 기존 shell 시간/P3 압력 공간 진단을 체크한다.
  서로 다른 backend/입력 범위를 합쳐 R1 acceptance로 승격하지 않는다.
- 공통 `DevProgressRow` 첫 인자 0/1이 완료 체크의 단일 값이며 합계도 여기서 계산한다.
  완료 시 본문 evidence/종료 근거와 상태를 갱신한 뒤 flag를 1로 바꾸고 PDF/bundle을 다시 빌드한다.
  Checkbox 개수는 개발 시간 또는 전체 작업량의 비율이 아니다.
- R2b 전체를 optional처럼 읽히게 하는 요약 문구는 재검토에서 바로 교정했다.
  R2a 뒤 R2b baseline·schedule/fallback 판정은 유지하고 period curriculum만 조건부다.

수정 파일은 sketch, master, R0–R7 및 shared preamble, 두 README와 기존 session/index,
직접 영향을 받는 PDF와 두 전달 bundle이다. 기존 unrelated dirty/untracked 파일과 다른 저장소는 보존한다.
완료 보고에는 실제 빌드·PDF 배치·링크·bundle 검증 결과를 함께 남긴다. 이번 변경은 commit/push하지 않는다.

### 실제 문서 검증 결과

수정한 canonical PDF 10개를 모두 실제 XeLaTeX/latexmk로 빌드했다. R0–R7과 master는 공통 preamble 변경을
반영해 각각 실행했고, sketch는 자체 preamble을 사용하는 별도 명령으로 빌드했다.
각 명령의 성공 종료, non-empty PDF 및 마지막 pass에서 undefined reference/citation, missing glyph,
overfull/LaTeX error가 없음을 확인했다. 최초 master 빌드에서 새 요약 문구의 밑첨자 표기 오류를 발견해
수학 표기로 고친 뒤 다시 빌드했으며 최종 결과는 모두 성공이다.

```bash
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  development/r0_contract_and_schema.tex
# 같은 명령으로 R1–R7의 owning source를 각각 빌드
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error \
  implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex
```

| PDF | 최종 쪽수 | 표지 다음 진행 페이지 |
| --- | ---: | --- |
| Sketch | 28 | 방법 문서이므로 개발 체크리스트 대상 아님 |
| Master | 7 | 2쪽 단계별 체크, 3쪽 목차 |
| R0 | 12 | 2쪽 장별 체크, 3쪽 목차 |
| R1 | 22 | 2쪽 장별 체크·개발 성과, 3쪽부터 목차 |
| R2 | 11 | 2쪽 장별 체크, 3쪽부터 목차 |
| R3 | 8 | 2쪽 장별 체크, 3쪽 목차 |
| R4 | 8 | 2쪽 장별 체크, 3쪽 목차 |
| R5 | 8 | 2쪽 장별 체크, 3쪽 목차 |
| R6 | 8 | 2쪽 장별 체크, 3쪽 목차 |
| R7 | 7 | 2쪽 장별 체크, 3쪽 목차 |

- 모든 개발 PDF의 2쪽에 8개 행과 종료 합계가 들어감을 텍스트로 확인하고 R1/master/R0의 실제 페이지를
  raster preview로 확인했다. 64개 장별 PDF 링크가 aux의 실제 본문 쪽수와 일치하며,
  master의 8개 외부 PDF 링크도 현재 파일을 가리킨다.
- R0–R7의 기존 8개 본문 장, acceptance checkbox 총 240개 및 기존 번호 있는 equation 본문을 보존했다.
  새 진행 표는 기존 acceptance 항목을 완료로 바꾸지 않는다.
- Sketch bundle 3개, master bundle 20개 entry가 모두 현재 source/PDF와 byte가 일치하고 ZIP 무결성 검사를 통과했다.
  새로운 별도 구현 기록이나 승인되지 않은 초안·빌드 중간물을 전달 묶음에 추가하지 않았다.
- 문서 Markdown 로컬 링크·whitespace와 작업 범위·기존 dirty 파일 hash를 검증했다.
  이번 작업의 변경은 ideas의 승인된 문서·PDF·bundle 27개에 한정되며 root/code/experiments는 변경하지 않았다.
  모든 저장소의 HEAD와 index는 그대로이고, commit/push 또는 Git 원격 fetch는 수행하지 않았다.

## 후속 실행: 변화 바람/velocity-reset의 실제 결과 통합

사용자의 “작업 계속해줘”로 이미 선택한 개발 비교를 수행했다. 설계만 남았다는 현재형 문구를
스케치/R1/master/index에서 실제 판정으로 교체했다. 기존 수식의 위치 보존/v0/kinetic 제거 계약은 유지한다.

- Native Newton demo/gravity-off의 별도 개발 schema. Teacher fixed-direction 계약이나 P3 model을 교체하지 않았다.
- 1 m XZ flag, x=0 pin, 질량0.1 kg, native stiffness1000/1000 N/m, damping0.1 s, bending10 N,
  kappa0.6, CPU/iterations10/fps60/90 frame. Seed20260909, 12 frame마다 target 생성/vector 선형 보간,
  마지막18 frame ambient0/air drag 유지, checkpoint18/42/66. Mesh4/8/16와 substeps8/16/32의5조건.
- `deterministic_prefix_replay_v1`로 solver 상태를 재구성하고 무개입 전체 replay와 reset prefix를 대조했다.
  빠른 내부 상태 직렬화가 아니며, 저장 checkpoint velocity는 개입 후 값이고 개입 전 값은 별도 배열이다.
- 25 trace/2,250 interval/2,275 state, CPU373.876초. 위치·속도·force/work replay 최대 차이0,
  pin/guard0, finite와 개입 energy/current-velocity 공력 재계산 통과. 신규8개+Newton 관련85개 검사 통과.
- Finest 자연 연속의 공간 변위0.520181%/속도12.040068%, 시간 변위0.077190%/속도2.176690%.
  모든 reset finest pair도 속도1% 진단 실패. 1.1 s reset의 공간81.148231%/시간17.756006%는
  절대 차이3.389095/0.555413 mm/s와 fine peak4.176425/3.128029 mm/s를 함께 보존했다.
- x=0 pin-line은 그 모서리 주위 강체 회전을 허용한다.0.7 s probe 변위 RMS12.700918 mm 중
  회전 fit 잔차0.002380 mm,1.1 s는22.540597 mm 중0.035869 mm다. 위치 분해이지 탄성 에너지 비율은 아니다.
  큰 position 변화가 충분한 내부 굽힘 상태 coverage라는 가정을 반례로 제한했다.
- 후속 후보는 기울기 또는 동일 물리 폭의 고정 영역을 제한하는 BC와 시간 오차를 먼저 낮춘 공간 진단이다.
  해상도마다 두 vertex 열을 고정해 물리 폭이 달라지는 조건을 같은 BC로 간주하지 않는다.
  이번에는 후보 BC를 구현하거나 새로운 data scope를 채택하지 않았다.

R1 5장의 계획 절을 설계+실제 결과로 갱신하고, 설정·재시작·검산·수치표·분모·회전 반례·재현 경로를
해당 위치에 통합했다. 6장 evidence/code session index,8장 채택 경계와 표지 다음 체크리스트도 갱신했다.
체크리스트에는 개발 비교 성과만 추가했으며 장 전체 종료0/8과 R1 미완료를 유지했다.
스케치는 초기 상태 분포의 주장을 이 증거 범위로 제한하고 master의 다음 조건을 갱신했다.

실험 원본과 compact evidence는 `experiments/R1_teacher_velocity_reset/README.md`가 소유한다.
Raw는 manifest 제외55개 파일/6,791,538 byte이고, 별도 프로세스의 재계산과 producer source25개 hash 대조를
수행했다. 대표 PNG 두 개는 재생성 시 byte-identical이며 회전 진단도 독립 재생성됐다.
학습 적격성 false, 새 학습 patch/window 또는 R2 학습을 발행하지 않았다. 기존15개 개발 window와 별도다.

이번에 수정한 canonical sketch/R1/master의 PDF를 모두 XeLaTeX/latexmk로 실제 빌드했다.
PDF/체크리스트/전달 bundle 검증 결과는 아래 최종 검증에 누적한다. Commit/push/fetch는 수행하지 않았다.

### 후속 실행 문서의 최종 검증

R2의 “다음 계획/미실행 reset” 문구도 실제 개발 실행과 학습 적격성 false로 동기화했다.
이번에 수정한 canonical PDF는 sketch28쪽, master7쪽, R1 23쪽, R2 11쪽이다.
Ideas root에서 다음 명령을 성공 실행했다.

```bash
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -g -xelatex -interaction=nonstopmode -halt-on-error \
  implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error development/r1_teacher_probe_oracle.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error development/r2_single_case_global_overfit.tex
```

모든 PDF non-empty와 최종 log의 undefined reference/citation·missing glyph·overfull/error 없음 확인.
Master/R1/R2의2쪽 체크리스트와3쪽 목차, 종료0/8 유지 확인. R1 체크리스트 렌더도 눈으로 확인했다.
두 canonical bundle의3개/20개 항목은 현재 source/PDF와 byte 단위로 동기화했다.
새 문서 링크, compact evidence hash와 개인정보 경로 제외를 확인했다.
작업 시작 전 snapshot과 대조해 root 및 네 repo의 무관한 dirty 파일을 보존했고 HEAD/index는 그대로다.
Code/experiments/ideas에 이 작업 변경이 미커밋으로 남아 있으며 commit/push/fetch는 하지 않았다.


## 후속: 고정 폭 굽힘의 실패 원인과 sample 발행 보류

사용자가 왼쪽0.25m 고정을 선택한 후의 초기25/pilot3/추가10 trace 결과를 sketch와 R1 본문에 흡수했다.
비평면 굽힘은 확보했지만 시간64→128의 변위/속도1% 동시 진단은 네 분기 모두 실패했다.
같은 해석적 굽힘 형상의 native 에너지가 mesh4→32에서 약84% 감소하는 정적 반례를 재현했다.
기존 native 감사 및 방향 편향으로 탈락한 면적 보정과의 관계를 명시했고 시간 refinement만으로 해결된다는
설명은 채택하지 않았다. 물리 오차의 전체 기여율이나 float32 원인을 확정하지 않는다.

사용자는 실패 개발 sample 발행보다 **원인 해결 후 생성**을 선택했다. 따라서 새 dataset 미발행,
학습 적격성 false, R1 미완료를 유지한다. 준비된12 interval/13 state·4patch 계약과 계획76/발행0의
구분, 중복 prefix/replay·reset 불연속 제외, 전역 work 비합산·전체 context·source group을 기록했다.
기존 P3의 작은 굽힘 연결과 Newton 비선형 굽힘 보완의 선택은 아직 미정이다.
새 BC/load의 검증을 기존 pressure 통과에서 승계하지 않고, 큰 변형 목표를 조용히 축소하지 않는다.

영향 파일은 sketch/R1/master의 source·PDF·두 bundle, ideas index/session이다.
Master와 R1 표지 다음 진행 문구를 현재 장애에 맞췄으며 완료 체크는 올리지 않았다.
새 연구 방향으로 pivot하거나 별도 R1 구현 기록을 만들지 않았다.

### 이번 후속의 최종 기록 검증

Sketch28쪽/master7쪽/R1 24쪽 PDF를 실제 XeLaTeX/latexmk로 빌드했다. 최종 log에 오류·미해결 참조·overfull이 없고, master/R1의2쪽 체크리스트0/8과3쪽 목차를 확인했다. 두 bundle3개/20개 항목의 source/PDF byte 동기화를 검산했다.
실험 evidence hash/producer snapshot 및 문서 링크, 네 저장소 diff 공백 검사를 통과했다. Root와 무관한 기존 dirty 파일의 hash 및 네 HEAD를 보존했다. Code/experiments/ideas는 현재 작업의 미commit 변경이 있고 새 commit/push/fetch는 없다.
현재 종료 상태는 물리 모델 선택 대기이며 sample 생성 완료가 아니다. 사용자가 선택하기 전에는 backend/물성 변경이나 실패 source의 sample 발행을 진행하지 않는다.


## 후속 완료: P3 선택·작은 굽힘 검증·sample76개를 canonical 본문에 반영

사용자가 기존 P3 활용을 선택했다. 작은 굽힘으로 범위를 제한하는 선택을 적용했으며 최종 큰 변형 목표를
이미 달성했다고 해석하지 않는다. Native 고정 폭 실패와 발행0 기록을 보존하고 새 P3 조건을 별도로 검증했다.

R1 초기 상태 절에 새 독립 소절을 추가해 다음을 직접 기록했다.
- 물성 E1e6Pa/nu.3/h.01m/면밀도.1kg/m², 자유 영역.075kg·고정 영역.025kg과 바람상한.05m/s.
- 자유 판 x=.25의 강한 변위/약한 slope 경계와 full boundary moment 식, 독립 spline 강한 slope 조건.
- Current graph normal/area, signed transpose force와 virtual power, 전체 모드의 exact held-force 식.
- P3 공간8→16 .882329%, 대각선/구적/spline refinement/독립 기준 모두1% 통과 및 coarse 실패 보존.
- 요소 내부/모든 시간의 Bernstein·spline convex hull+modal amplitude envelope, kinetic reset과 work ledger.
- 두 full run의36trace,618개 배열/44,291,978scalar·물리 report 일치와 원본·source snapshot·그림 경로.
- 실제19window/76patch, sample 소비 계약과 원본/NumPy batch 검증,11개 검사 및 제한된 적격성.

Sketch의 미정/발행 보류를 사용자 선택과 실제 결과로 갱신했다. Master/R1 앞쪽 체크리스트에는
P3 sample76개 검증 성과를 반영하되0/8 단계/장 완료는 유지한다. R2도76개 sample의 범위를 설명하고
oracle/GS·student 초기화 선행 조건을 유지한다. 별도 R1 구현 기록으로 다시 분리하지 않았다.
원래 pressure의 position-only 식/결과와 새 clamped 작은 굽힘의 wind 식/결과를 같은 조건으로 합치지 않는다.

공식 FEniCS-Shells clamped KL 경계식을 웹으로 확인했고 URL은 실험 README에 남겼다.
이번 작업은 Teacher의 제한된 샘플 범위 보완이며 현재 target response-package 방법의 pivot이 아니다.
PDF·bundle·체크리스트 및 변경 범위 검증은 아래에 누적한다.


### P3 후속 PDF·전달 묶음·상태 확인

수정한 sketch28쪽, master7쪽, R1 26쪽, R2 11쪽 PDF를 실제 XeLaTeX/latexmk로 빌드했다.
Ideas root에서 각각 다음 명령을 사용했다.

```bash
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error 3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error development/r1_teacher_probe_oracle.tex
latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error development/r2_single_case_global_overfit.tex
```

R1 종료 표의 현재 sample 상태를 정리한 후 동일 명령에서 `-g`를 뺀 추가 빌드를 수행했다.
네 PDF가 non-empty이며 최종 log에 overfull·undefined reference/citation·missing glyph·LaTeX 오류가 없다.
Master/R1/R2의2쪽 체크리스트0/8과3쪽 목차, 모든 새 PDF의76개/P3 반영을 추출 텍스트로 확인했다.
Sketch bundle3개와 master bundle20개 항목의 byte 동기화를 검산했다. R0/R3–R7 source와 기존 무관한 초안은
수정하지 않았다. Ideas 변경은 미commit이며 새 stage/commit/push/fetch는 없다. 이번 범위의 sample 목표는
완료했지만 작은 굽힘 밖의 일반화나 R1 전체 완료 체크를 올리지 않는다.


## 후속 지시: 물리 수식 우선과 P3 3D 요소 연산

사용자는 물리 수식을 제대로 구현한 뒤 학습데이터를 생성하도록 지시했다. 추가 발행은 보류하고
76개 작은 굽힘 sample은 과거 개발 근거로 보존한다. 연구 target 방법을 바꾸는 pivot은 아니다.

- 판정/근거: 작은 굽힘 판의 통과가 최종 finite rotation/membrane+bending 완료가 아니라는
  기존 한계를 명시적으로 유지한다. P3를 재사용한 3D 기하·질량·가상 일의 요소 연산을 보완했다.
- 수식: R1 구현 장의 `sec:r1-p3-surface-equations`에 현재 metric/곡률, normal의 위치 미분,
  내부 가상 일의 음의 adjoint, current-area relative wind와 passive work를 직접 통합했다.
  재료 응력 생성과 전역 에너지 Hessian까지 구현했다고 표시하지 않는다.
- 검증: 신규 10개, 기존 포함 21개 회귀 검사 통과. 기존 P3와 4개 정적 대조/240개 요소의
  mass/force/work 최대 상대 차이는 1.76425e-15 미만이다. 새 학습데이터는 0개다.
- 문서 영향: sketch의 과거 pressure-only 한정 문구를 해당 과거 fixture로 명확히 했고
  이미 검증한 작은 굽힘 wind와 구분했다. R1의 구현 표·수식·미결정 표와 master/R2 진입 경계를
  최신 지시에 맞췄다. R1 전체 체크나 다음 단계 완료 체크를 올리지 않는다.
- 아직 미결정: 유한 회전 shell(권장), von Kármán 판, 선형 P3의 장단점을 질문했다.
  답변을 받기 전에는 최종 nonlinear 구조식·element coupling을 동결하지 않는다.
- 외부 근거: Verhelst et al. (2024)의 Kirchhoff–Love 기하와 virtual work 원문을 웹으로 조회했다.
  URL은 실험 README에 있으며 두께 방향 strain의 곡률 부호와 현재 b=n·H 정의를 구분했다.
  코드 다운로드/Git fetch는 수행하지 않았다. 근거 논문의 spline acceptance를 P3에 승계하지 않는다.

PDF 및 bundle의 실제 빌드·검산 결과는 아래에 덧붙인다. Ideas 변경은 미commit·미push다.

### 이번 후속 PDF·bundle 검산

Ideas cwd에서 앞 절과 같은 `latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error`
명령으로 sketch/master/R1/R2 네 문서를 각각 실제 빌드했다. 모두 성공 종료했으며 PDF는
28/7/26/11쪽, non-empty다. 최종 log에 overfull·undefined reference/citation·missing glyph·LaTeX 오류가 없다.
R1의 새 4.2.3 요소 수식, sketch의 후속 진행 원칙, master/R2의 추가 발행 보류가 PDF 본문에 있다.
2쪽 체크리스트와 3쪽 Contents 시작을 유지하며 기존 미완료 체크를 올리지 않았다.
두 canonical bundle을 각각 3/20개 source/PDF 항목과 byte 대조해 동기화했다.

검산 도구의 목차 문자열을 한국어로 가정한 assertion은 기존 PDF의 실제 `Contents`에 맞춰 고쳤다.
Bundle 교체의 `/tmp`→workspace rename은 filesystem이 달라 EXDEV가 발생하여 같은 ideas filesystem의
임시 파일에서 검증 후 atomic replace했다. 문서 내용이나 수치 기준을 낮춘 조치는 아니다.
Root와 R0/R3–R7 및 기존 무관한 초안은 변경하지 않았으며 Git HEAD는 네 저장소 모두 유지했다.


## 후속 사용자 결정: 1번 유한 회전 shell 채택 방향

사용자가 **1번 유한 회전 shell**을 선택했다. 이는 기존 P3를 재사용한 Teacher의 구현 범위 결정이며,
Global/Local response 방법의 research pivot은 아니다. 기존 sample과 실패 모델의 원본은 보존한다.

- 판정/근거: finite rotation/membrane+bending의 최종 목표에 맞춰 Koiter/StVK flat-rest shell을 구현한다.
  큰 회전을 허용하되 작은 재료 strain·고정 두께의 thin-shell 범위를 유지한다. 접촉/소성/두께 변화는 추가하지 않는다.
- 수식: R1의 `sec:r1-p3-shell-equations`에 volume/edge energy, current normal의 전체 미분,
  full clamped boundary flux, 고정 법선 frame couple, consistent M와 nonlinear Newmark를 직접 통합했다.
  선형 P3의 normal-slope 경계에서 누락된 고정 반력 행 미분과 수정 이유도 수식 옆에 남겼다.
- 검증/한계: 정적 원통36개, independent DOP853 short response, actual wind/rest/reset 및
  bent release를 구분한다. 정적 수렴이나 실행 완료만으로 동적 공간·장기 안정성을 완료 처리하지 않는다.
  Actual wind n8→16의1.357196%, 초기 굽힘 release sub128→256의13.313118% 속도 차이는 실패로 보존한다.
- 문서 영향: sketch의 선택 대기 문구를 교체하고 R1 본문/미결정 표, master 및 R2 진입 경계를 동기화했다.
  모델 선택은 해결되었지만 oracle/GS·채택 범위·수렴 조건은 별도로 남기며 R1/R2 완료 체크는 올리지 않는다.
- 원문 근거: Noels (2009) nonlinear DG Kirchhoff–Love shells의 normal jump/consistent moment flux와
  대조했다. 고정 두께 Koiter 에너지의 보존적 변분이며 논문의 hyperelastic 두께 적분이나 nonlinear 안정성
  결과를 구현·승계했다고 주장하지 않는다. Primary URL은 R1와 실험 보고서에 있다.

PDF는 sketch/master/R1/R2를 실제 XeLaTeX로 빌드한다. 첫 빌드의 circled digit missing glyph는
TeX 문구를 일반 “1번”으로 바꾸어 해결한다. 새 물리 수식이 있는 R1을 별도 기록 문서로 분리하지 않는다.
후속 검증 수치와 최종 PDF·bundle 검산은 이어 기록한다. Git stage/commit/push/fetch는 없다.

### 수식·수치 해석 보완과 논문용 근거 범위

R1 본문의 기존 P3 기하 절 다음에 선택된 유한 회전 구조식을 통합했다. 후속 검산에서 구적점의
면적 검사와 별도로 Bernstein 공간/시간 상한을 추가했으므로, 기존의 “전체 요소는 미검사” 문구도
선행 geometry guard에 한정하고 새 절로 연결했다. 정확한 연속 ODE나 interval arithmetic 인증을
얻었다고 표현하지 않는다. Small material strain/linearized through-thickness 범위도 명시했다.

약한 바람의 공간 n16→32 속도 .385696%, n16 대각선 .153774%와 원통 release의 n4 시간
sub1024→2048 .762185%를 기록했다. 원통의 자유단 moment 불일치와 초기 접선 진단을 함께 적었으며,
거친 공간 실패를 바람으로 도달한 상태의 성공으로 덮어쓰지 않는다. Short DOP853은 같은 ODE의
독립 시간 검증이고, 아직 finite-shell 독립 공간 reference가 아님을 유지한다.

원래 요청한 rest-start/reset은 별도5m/s peak의3 frame 진단에서 실제 약7.3cm 변형을 만들었다.
N8 시간 차이는.304620%, n8→16 공간 차이는1.849529%여서 후속 세분화를 진행한다.
마지막 frame은 ambient0/reset velocity0으로 held force0이므로 탄성에너지의 자유 응답이며,
유한 변형 중 방향 변화 바람은 앞선 두 frame에 해당한다. 데이터 풍속 범위를 동결한 결과는 아니다.
실험 README에는 실제 중앙 단면과 운동/탄성/총에너지/reset의 PNG·벡터 PDF를 붙여 의미를 설명했다.

R1의 수식 검증 문구는 kernel/solver15개+validator5개+Bernstein3개=신규23개 통과로 갱신했다.
최종 동적 refinement 수치, PDF page/layout와 bundle 동기화는 종료 검산 절에 누적한다.

### GPU 이식 선택과 미분식·수치 보간 근거 통합

사용자는 장기 계산 비용을 줄이는 다음 조치로 GPU 이식을 선택했다. Koiter/StVK 에너지,
full-normal 경계, consistent mass, 60Hz nodal-force hold와 Newton/Newmark tolerance는 유지한다.
첫 이식은 Warp float64 연산자이며 Newton/희소 풀이와 마지막 에너지 합산은 CPU다.
Target response package·학생 모델·물성·풍속 범위 또는 학습 curriculum을 변경한 결정은 아니다.

R1의 같은 구조식 절에 다음을 추가했다.

- Normal의 정확한 방향 미분, volume gradient와 edge의 tangent/moment/normal 전체 gradient를 명시했다.
  CPU/GPU가 이 식을 함께 사용하고 dual value/direction을 전파해 HVP를 얻는 방식을 적었다.
- N16 wind-reached 형상의 평면 내 변위7.23639mm를 제거하면 막 에너지가7.06081e-5J에서
  .902094J로 변하는 실제 상태 대조를 넣었다. 같은 law의 설명용 대조이며 선형 P3 재실험은 아니다.
- Common fine clock와 reset 좌극한을 포함한 수치 보간 응답 상한을 유도했다.
  저장 u/v의 kinematic defect까지 포함하는 변위 Lipschitz 여유와 reference denominator 하한을 사용한다.
  원식 raw 검산과 정확한 연속 ODE/interval arithmetic 인증을 구분했다.
- 약한 바람의 n32 시간 상한.231300%, n16→32/sub256 공간 상한.381161%를 확인했다.
  강한 바람의 n16→32/sub128 속도 상한.3917943%, n32 시간 상한.2437471%도 통과했다.
- 실제 GTX1080Ti의12개 연산자 상태, CPU/GPU full trajectory와17개 배열의 독립 replay를 연결했다.
  Source snapshot, 로그·실패 보존과 성능 측정 범위는 실험 보고서가 소유한다.

수식 파일만의 묶음에서 빠진 package 초기화 import를 보완하기 위해40개 runtime bootstrap을 따로 보존했다.
이는 source22개 identity와 제3자 dependency environment를 대체하지 않는다.
R1 체크리스트의 개별 개발 성과와7장 남은 조건을 최신 상태로 바꿨지만, 장 종료 체크는0/8을 유지했다.
Master의 R1 상태도 GPU 개발 검증과 장기/reference 채택·추가 데이터 보류를 구분한다.

검토 PDF는 sketch29/master7/R1 30/R2 11쪽으로 실제 빌드했다. R1의11쪽 gradient 식과2쪽 체크리스트를
이미지로 확인했고 master/R1/R2의2쪽 체크리스트·3쪽 Contents 배치를 확인했다.
최종 refinement 수치 반영 후 PDF와 두 bundle의 최종 byte 대조는 아래 종료 절에 기록한다.

### 단기 수렴과 다음 바람 범위의 사용자 결정 반영

R1 본문에 strong n16→32/sub256 공간 속도 상한.4000651%, n32 시간 상한.2437471%,
방향 상한.0406189%의 표를 추가했다. Weak finest 방향 상한.0397466%, Bernstein 기하·strain과
격리 source 재실행까지 본문에 흡수했다. Sketch에도 단기1% 통과와 채택 보류 경계를 반영했다.
원래 느린 CPU n32 strong 및 최종 v5 wind24개/비교20개도 완료됐으며 CPU/GPU 대조를 통과했다.

사용자는 장시간 풍속을 **기존0.25–0.5m/s부터 검증 후 확대 여부 판단**으로 선택했다.
R1/sketch에1.5초 rest natural,0.3/0.7/1.1초 위치·시각 보존/velocity0 분기와 동일 미래 바람을 명시했다.
R1에는 parent checkpoint의 재사용·identity, 전 interval 검산 및 CUDA evaluator의 CPU 대조 범위를
추가했다. 선택한 knot 범위와 보간된 실제 풍속을 구분하고5m/s 단기 진단을 데이터 범위로 승격하지 않는다.
두 문서의 변경을 실제 XeLaTeX로 빌드했으며 undefined/overfull 오류는 없었다.
긴 구간 진행 결과와 최종 bundle 동기화는 아래에 이어 기록한다.

### 초기 장시간 결과·정규화 기준의 본문 통합

R1과 sketch에1.5초 natural n8→16 공간 속도 상한.699080%, n8 natural+세 reset의29,952 interval
원식·에너지·기하 검산, 최대 변위 약9.1mm를 반영했다. R1은 제거 kinetic과 재시작의 정확한 배열 수,
zero ambient의 음의 누적 일도 명시하며 남은 reset/시간/방향 및 큰 변형의 적절성을 분리했다.

RMS 점검에서 처음 단위 오류로 해석한 설명은 선행 README의 전체1m² 평균 정의를 확인한 뒤 정정했다.
R1의 초기 가속도는 기존 전체 영역 RMS368.481/1005.274/2793.453m/s²와
rest wind1.28553/1.29226/1.29564m/s²를 유지한다. 새 장시간 비교의 자유0.75m² 평균과
변환인자1/sqrt(.75)를 별도로 정의한다. 상대 오차와1% 판정, 물리 원본에는 변화가 없다.
이 해석 과정도 본문 주석과 실험 sidecar에 남겨 논문 작성 때 정규화 영역을 혼동하지 않게 했다.

중간 bundle은 기존3개/20개 member를 유지해 당시 source/PDF와 byte 단위로 맞췄다.
이후 본문 변경의 PDF 재빌드와 최종 bundle 재동기화는 아래에서 확정한다.

### 1.5초 약한 랜덤 바람의 최종 검증 완료

사용자가 선택한0.25–0.5m/s knot, 기존 seed/방향/시각/물리 law를 유지했다.
N8/sub128, n16/sub128, n8/sub256, n16/sub256 forward/backward의5조건에서 natural과
세 독립 reset을 끝냈다. Primary20개 원본/239,616 interval/1,170 frame의 원식·에너지·고정 조건·
Bernstein 검산을 통과했다. CUDA 전 interval 검산과 별도로 CPU 상태3,510회(경계 중복 포함),
공력1,170회를 대조했다. Checkpoint5개·격리 재시작·smoke는 이 primary 합계에서 제외한다.

| 속도 수치 보간 상한 (%) | Natural | Reset18 | Reset42 | Reset66 |
|---|---:|---:|---:|---:|
| 공간8→16/sub128 | 0.699079818 | 0.910952981 | 0.848094674 | 0.720627912 |
| 공간8→16/sub256 | 0.694414320 | 0.904299033 | 0.842757344 | 0.711992708 |
| 시간128→256/n8 | 0.108164309 | 0.136702936 | 0.119953292 | 0.169299080 |
| 시간128→256/n16 | 0.096663827 | 0.119147052 | 0.096074463 | 0.148473407 |
| 방향/n16/sub256 | 0.044940356 | 0.055049300 | 0.045901233 | 0.063805441 |

20개 비교의 변위/증분/속도 모두 기존1%를 통과했다. 변위 최대.071417445%, 증분.072494201%다.
공간 차이가1%에 가까워 n8/sub256을 추가했으며, 시간 간격을 절반으로 줄인 후에도 공간 최대 속도
상한.910952981%→.904299033%로 유지됐다. 이번 약한 바람 범위에서는 n32 장기 계산을 추가하지 않았다.
허용오차·수식·재료·기준을 완화하지 않았다. 독립 비선형 공간 기준의 통과로 승계하지 않는다.

Finest n16/sub256 natural의 최대 nodal 변위9.067314262mm, 면적비 하한.99971002732,
mid-surface strain 상한3.580886766e-6, 선형 표면 strain 상한.000308814685다.
Reset18/42/66 제거 kinetic은4.196988109/7.363603640/16.355722756µJ다.
Finest natural 마지막0.3초 외력 일은−1.378832651µJ이며20개 원본의 tail 모두 음수다.
추가 구조 감쇠 없이 상대풍 drag가 순 에너지를 제거했다. 전체 힘 잔차/허용오차 최대 비는.0238135532다.
다섯 checkpoint 재시작은9개 배열과 step diagnostics가 정확히 같았다.
Sub256 n8/n16의 scalar 수는735,416/2,802,626개다.

도달한 위치·시각을 보존하고 속도만0으로 제거한 뒤 같은 미래 바람으로 이어가는 절차는
이번 약한 바람에서 수치적으로 성립했다. 약9mm 변위이므로 큰 변형 장기 coverage는 남는다.
다음 풍속 확대는 사용자 선택을 받은 뒤 시작하며, 추가 학습데이터는0개다.
개발 검증 true와 training_eligible/r1_complete false를 함께 보존한다.

R1 본문에20개 비교 속도 상한 표, 변위/증분·reset 에너지·zero ambient 일·재시작 scalar 수·
검산 범위와5상태 구적 결과를 흡수했다. Sketch에는 최종 수치와 큰 변형/독립 공간 기준의 한계를 반영했다.
R1 표지 다음 체크리스트에 약한 바람1.5초의 개발 성과를 체크했으며 장 전체 종료0/8은 유지했다.
Master의 진행 상태도 약한 바람 검증과 큰 변형/reference 채택 보류를 구분한다.

최종 XeLaTeX/latexmk 실행은 R1/sketch/master 모두 exit0이며 오류·undefined·overfull·missing character가 없다.
R1 31쪽, sketch29쪽, master7쪽을 생성했다. 이번 작업에서 앞서 수정한 R2도11쪽 빌드를 완료했다.
Master/R1/R2의2쪽 체크리스트·3쪽 Contents, R1의2쪽 체크리스트와14쪽 결과 표를 확인했다.
Source/PDF를 두 bundle의 기존3/20개 member와 byte 단위로 맞췄다.
Sketch bundle SHA `dc10f891165abfd8c57d74c3e81dae4bcb2c79864421b064d90eb4eec757cf18`.
Master bundle SHA `eb63f4ca5e81b45c368de97043e765b9397e63d93291d242a88cbd174e4340fc`.
과거 사용자 소유 R0/R3–R7 TeX는 수정·재빌드하지 않았다.

### 최종 보존 확인과 다음 결정 대기

최종 무결성 확인을 실제 실행해 통과했다. Runtime v4의47개 source와 현재 파일·ZIP member,
실험 wrapper3개와 snapshot, 문서 bundle3/20개 member의 byte 일치를 확인했다.
CPU/GPU/긴 랜덤 바람의 작은 근거 inventory147/100/205개가 각각 SHA와 크기에 맞았다.
기존 작은 굽힘 sample99개 output/376,077 byte와 원본54개 output/167,594,035 byte의
manifest 및 모든 output 해시도 이전 보존 기록과 같았다. 공유 후보 텍스트585개에서 개인 절대 경로를 발견하지 않았다.

Root는 작업 시작 전 상태를 그대로 보존했다. Code는 구현·문서, ideas는 canonical source/PDF/bundle·기록,
experiments는 실행/검산·그림·작은 근거·기록의 미커밋 변경이 있다. 네 저장소 HEAD와 기존 무관 변경은
보존했으며 diff --check를 통과했다. 이번 작업의 stage/commit/push/fetch는 모두0건이다.
원격 최신성은 조회하지 않았고 origin 비교는 로컬 remote-tracking ref에 한정한다.

다음 사용자 질문을 제시했다: 동일 방향·시각의 바람 파형을4배(목표 knot1–2m/s, 권장)로
단계적으로 키울지,10배(2.5–5m/s)로 바로 큰 변형을 확인할지 선택한다.
전자는 실패 원인 추적이 쉽지만 변형이 부족하면 추가 확대가 필요하다. 후자는 큰 변형을 적극적으로
확인하지만 수렴 실패·계산 비용 위험이 크다. 답변을 받기 전에는 새 풍속 실행을 시작하지 않는다.
약한 바람의 완료 검증과 다음 확대 선택을 분리하며 학습데이터 추가 생성은 계속 보류한다.

### 사용자 승인: 4배 파형의1–3단계

사용자는 제시한 다음 작업의3번까지 진행하도록 요청했다. 권장한 기존 파형4배, 목표 knot1–2m/s에서
1.5초 rest natural과0.3/0.7/1.1초 독립 velocity-reset, 원식·공간/시간/방향1% 검증을 수행한다.
Seed·방향·시각·고정 폭·재료·물리 law와 허용오차는 유지한다. 이 응답으로 앞선 풍속 선택 대기를 해소했다.
4번의 후속 확대·재료 및 학습 적격성 채택, 학습데이터 생성은 이번 승인 범위에서 제외한다.
수치 결과가 완료되면 같은 R1/sketch의 해당 절에 근거와 적용 범위를 반영한다.

### 4배 범위의 문서 반영과 첫 공간 실패

승인된1–3단계를 sketch/R1/master와 canonical index에 반영했다. Sketch29쪽, R1 31쪽,
master7쪽의 실제 XeLaTeX/latexmk 빌드가 성공했고 새 overfull/undefined/missing-character 오류는 없다.
3/20 member 전달 bundle은 기존 member set을 유지한 채 모든 member를 현행 파일과 byte 대조했다.
이 중간 PDF는 승인 범위를 반영한 상태이며 이후 최종 검증 수치는 완료 후 갱신한다.

첫 n8→16/sub128 natural의 속도 상한1.087961809%는 기준1%를 넘었다.
변위/증분0.0632761324%, 최대 속도 차이는 후반 frame88이다. 두 natural의 원식 검산은 통과했다.
물리식/허용치 변경 없이 n32/sub256와 n16→32 비교를 추가한다. 후속 범위 확대·학습 적격성 채택은 하지 않는다.

### 진행 중인4배 검증의 canonical 본문 반영

R1에4배 입력의 동결 계약, schema v2/배율 identity, runtime v5/source47와 이전v4 재현,
새 관련검사10개/실제CUDA·CPU smoke, 완료된 n8 전분기와 n16/sub256 natural 검산을 기록했다.
공간 natural/reset18/reset42의1.087962/1.211015/1.193883% 미달과 n16 natural 시간/방향
0.144732/0.059589% 통과를 표로 구분했다. 최대변위109.084864mm와 선택5상태 구적,
오차법선성분98.6821%, 같은기준의 n32 보완과 미완료 경계도 반영했다. Sketch 관련절도 동기화했다.

두 canonical PDF의 실제 latexmk/XeLaTeX 빌드가 성공했다. R1 31쪽/Sketch29쪽이며
new overfull/undefined/missing-character 오류는 없다. PDF텍스트에서 새 실패 수치를 실제확인했다
(R1 15쪽,Sketch10쪽). 기존3/20member의 bundle은 모든 member를 현재파일과 byte 대조해 갱신했다.
Master source/PDF는 이미4배 검증중으로표시하며 이 추가본문반영에서 다시수정하지 않았다.
최종fine결과가완결되면 이 진행상태를 갱신하고 영향받는PDF/bundle을 다시검증한다.

R1 새 결과 page15를 실제이미지로 확인했다. 기존\code의nolinkurl이 CLI공백을 생략해 --wind-scale4로 보이는 문제는 새호출을분리하여수정했다. R1 PDF를재빌드하고 pdftotext에서 --wind-scale 4를확인했다. 관련오류없음, master bundle20member도재대조했다.

Coarse 공간 네번째분기(reset66)의3.495261% 미달도R1/Sketch에추가했다. 절대속도오차1.87151mm/s와reference53.5442mm/s를함께설명하고1%기준을유지했다. 두PDF실제재빌드/새수치텍스트확인/overfull등오류없음 및3/20memberbundlebyte대조도완료했다.

### N16 시간·방향 전체 비교와 N32 첫 원식 검산 반영

4배 바람의 n16 시간 비교 네 분기와 방향 비교 네 분기가 모두 기존 1%를 통과했다. 최대 속도 상대 상한은 시간 0.73183011529917%, 방향 0.27452723457168475%다. 처음 네 조건의 주원본 179,712 interval과 checkpoint 768 interval 검산이 모두 완료됐다. N32/sub128 natural의 11,520 interval 및 checkpoint 128 interval 검산, 9개 배열 / 5,502,710개 scalar와 step diagnostics의 정확한 재시작도 확인했다. N32 자연 응답 최대 nodal 변위는 109.09146016523852mm다.

R1 표의 n16 세 reset 칸을 실제 결과로 교체하고 원식 검산 범위를 본문에 추가했다. Sketch도 같은 판정으로 갱신했다. N32의 세분 비교는 계속 진행 중이며 coarse 공간 네 실패와 1% 기준, 학습데이터 0 경계를 유지했다.

실제 XeLaTeX/latexmk 빌드 성공: R1 32쪽, Sketch 29쪽. 새 최대값 0.731830/0.274527의 PDF 텍스트를 확인했고 overfull/undefined-reference/LaTeX 오류는 없다. 두 전달 bundle의 기존 3개/20개 member를 현재 파일과 byte 대조해 갱신했다. Master는 기존의 4배 검증 진행 표시를 유지하므로 이번 중간 갱신에서 source를 다시 수정하지 않았다.

R1 갱신 결과 15쪽을 PNG로 렌더링해 실제 확인했다. coarse 네 실패의 굵은 표시와 n16 여덟 통과의 표, 검산 범위·보완 미완료 문장이 페이지 안에 정상 배치되어 있다.

### 첫 n32 공간·시간 보완과 원식 검산의 canonical 반영

N16→32/sub256 natural의 공간 비교가 속도 0.4441317011175531%, 변위/증분 0.015113537118286485%로 통과했다. N32 sub128→256 natural 시간 비교도 속도 0.14017320185221683%, 변위/증분 0.000776833622689169%로 통과했다. N32/sub256 natural의 23,040 interval 원식·기하 검산을 완료했고 최대 nodal 변위는 109.09161545556276mm다. Bernstein 면적비 하한 0.9603993022634437, mid-surface/fibre strain 상한 0.00029430928983860063/0.003948190025140212, 최대 frame 에너지 결산 차이 7.517887918993388e−12J, tail 일 합 −0.0007659055302491345J를 R1에 반영했다.

R1의 표에 n32 관련 세 비교 행을 추가하고 실제 완료한 natural 공간·시간 수치만 채웠다. 나머지는 미완료 표시를 유지한다. Sketch에도 같은 범위를 통합했다. Coarse 공간 비교는 sub128이고 보완 비교는 sub256이므로, 두 값만으로 공간 수렴 차수를 추정하지 않으며 별도 n16/n32 시간 비교를 제시한다는 한계를 추가했다.

두 canonical PDF를 실제 XeLaTeX/latexmk로 빌드했다(R1 32쪽, Sketch 29쪽). 0.444132/0.140173의 PDF 텍스트를 확인했고 overfull/undefined-reference/LaTeX 오류가 없다. 전달 bundle의 기존 3개/20개 member를 모두 현재 source/PDF와 byte 대조했다. 빌드 후 n32/sub256 checkpoint42도 9개 배열/10,941,686개 scalar와 step diagnostics가 정확히 일치하고 256 interval 검산을 통과했다. 이 추가 재시작 상세는 raw 보고서와 진행 로그에 보존하고 최종 결과 통합 시 R1에 함께 정리한다.

R1 새 15–16쪽을 실제 이미지로 확인했다. 6행 비교표는 15쪽 안에 정상 배치되며, 후속 재시작/미완료 설명이 16쪽으로 이어진다. 완료와 대기 칸이 구분되고 겹침이나 잘림은 없다.

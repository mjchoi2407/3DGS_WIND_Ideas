# 2026-09-09 01 Teacher 전체 개발 근거의 canonical 문서 통합

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
| 첫 accepted 입력군 | 보류 | 사용자에게 범위를 질문했으나 선택 미확정 | Rest-start 우선 여부 결정 전 기존 displaced decay 요구 유지 |
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

P3 nonlinear wind backend/consistent mass/고차 map의 구체적인 설계와 입력 범위 결정부터 이어간다.
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

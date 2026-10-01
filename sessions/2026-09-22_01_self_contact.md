# CPU/GPU 셀프 접촉 후보의 R1 상태 반영

## 현재 상태

- 2026-10-01 사용자 정리·push 요청에 따라 [R1](../development/r1_teacher_probe_oracle.tex)의 `sec:r1-cg-wind-damping-status`에 v13 완주, 감쇠24 시간 비교와 새 내부 감쇠·바람 선택의 적용 범위를 반영했다. R1 완료 체크·Gate는 유지한다.
- 현재 선택은 P3 강성1/500·막5ms/굽힘20ms·전역0·기하 재사용과 시간 평활 균일 바람이다. [감쇠 추천·비용 근거](../../experiments/R1_teacher_velocity_reset/self_contact/vibration_search.md#최종-추천과-비교), [사용자 외력 선택·한계](../../experiments/R1_teacher_velocity_reset/self_contact/wind_field.md#사용자-시각-선택과-후속-외력-기준)를 따른다. 국소 바람은 미채택이며 솔버 원인을 배제하지 않는다.
- [감쇠24의 같은 장치64/128 수치 통과](../../experiments/R1_teacher_velocity_reset/self_contact/damping24_time.md#서브-완료-결과-분석)는 새 조합에 승계하지 않는다. 새 조합의 전체 궤적·시간/공간 민감도·GS/작은 응답·학습은 미판정이다.
- Sketch/master/R1/R2 PDF 빌드와 두 전달 묶음 동기화를 완료했다. [정리 검증](#2026-10-01-정리-검증)의 범위를 따르며, 기존 사용자 TeX 초안3개와 ignored 실행 원본은 제외·보존한다.

## 2026-10-01 정리 검증

- 사용자 요청의 누적 변경을 독립 저장소별 commit/push 대상으로 정리했다. 기존 CG 개발 진입 계약과 이번 R1 결과 동기화를 함께 포함하며, 최종 revision은 각 저장소 Git 이력을 따른다.
- Sketch/master/R1/R2를 XeLaTeX/latexmk로 실제 재빌드했다. PDF는 각각31/7/40/11쪽이고 최종 로그 경고·미해결 참조0개다. R1 완료 체크8개와 미완료 항목38개의 상태를 보존했다.
- Sketch/master 전달 ZIP의3/20항목 무결성 및 현재 파일과의 byte 일치를 확인했다. 이번 빌드 전 산출물은 로컬 임시 복구본으로 보존했다.
- 관련 구현 회귀256개와 명시적 GPU 검사10개(그중2개 중복)가 통과했다. Python64개·JSON24개·셸36개의 구문 검사, 변경 Markdown의 로컬 링크666개를 확인했다. 이는 구현 회귀 검증이며 새 전체 시뮬레이션·학습 완료 근거가 아니다.
- 서브컴은 유휴였으나 pytest/latexmk가 없어 기존 메인 환경에서 검증·빌드했다. 네 원격의 fetch 후 기존 HEAD 일치를 확인했고 외부 패키지 다운로드·설치는 하지 않았다.

## 2026-09-28 비교기·CG 진입 반영 당시 상태

- 2026-09-28 공통512점 비교기 준비를 R1 `sec:r1-cg-comparator-ready`에 반영했다.
  [소유 명령·검증·한계](../../experiments/R1_teacher_velocity_reset/self_contact/cg_comparator.md#현재-상태).
  CPU29개 검사·v12 자기 비교는 도구 검증이며 실제 두 정밀도의 민감도 통과가 아니다.
  수치 pass는 시각 대기, 브라우저 재생은 미검증이고 T/S runner·24 지원·GS/학습·정식 Gate는 미완료다.
  R1 PDF40쪽 최종 로그 경고 없음. master bundle20개 중 R1 source/PDF2개만 교체하고18개와 완료 체크를 보존했다.
  변경 전 복구본은 `.latex-build/cg_comparator_before_20260928/manifest.json`에 있다.

- 2026-09-28 CPU 처짐 분석·공통512점 P3 위치/속도 매핑을 구현하고13개 검사·v12 세 씬 실결과로 검증했다.
  [소유 결과·정적16/32 map·24 미지원 한계](../../experiments/R1_teacher_velocity_reset/self_contact/motion_analysis.md#구현과-검증).
  R1 `sec:r1-common-motion-analysis`와 개발 진입 절의 구현 상태만 갱신했다. 비교기·GS/작은 응답·학습 적격성은 미완료다.
  R1 PDF40쪽 빌드·최종 로그 경고 없음, master bundle의 R1 source/PDF2개 동기화·나머지18개 보존을 확인했다.
  변경 전 복구본은 `.latex-build/motion_mapping_before_20260928/manifest.json`이며 완료 체크는 유지했다.
  서브 컴 v13 계산은 사용자 보고다. 이번 작업은 CPU 분석이며 실행 중 작업·동결 runtime을 수정/중단하지 않았다.

- 2026-09-28 사용자 요청으로 CG 개발용 두 해상도 민감도·시각/매핑 검사 후 제한 학습을 채택했다.
  기존 전면 학습 보류는 이 제한 범위에서 대체하며 정식 R1/R2와 Gate 완료 조건은 유지한다.
- 기준 소유: [R1](../development/r1_teacher_probe_oracle.tex) `sec:r1-cg-development-entry`의
  `cg_teacher_dev_v1`(위치1%L·속도10%·시각·기존 검산). 제한 학습 소유:
  [R2](../development/r2_single_case_global_overfit.tex) `sec:r2-cg-development-entry`.
- [sketch](../3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex)의
  `sec:rd-cg-development-entry`와 [master](../implementation_checklist_response_distilled_global_local_wind_dynamics_2026-08-22.tex)의
  현재 상태·R2 진입을 동기화했다. 공식 claim·수식·solver 허용오차는 유지한다.
- 대표 사각형의 신규 실행2개·합계6시간, v12 뷰어→시간→공간→GS/작은 응답 확인 순서다.
  [정확한 설정·명령·미구현 범위](../../experiments/R1_teacher_velocity_reset/self_contact/cg_development_checks.md#현재-상태).
- 후속 v12 뷰어 연결·표시 검증을 완료했다. [명령·검증 범위](../../experiments/R1_teacher_velocity_reset/self_contact/three_scenes_gpu_v12.md#연속10초-뷰어).
  비교/실행 래퍼·사용자 시각 판정·새 CG 검사·개발 학습은 미완료이고 기존 v12 eligibility는 유지한다.
  R1의 개발 진입 조건을 확인했으며 표시 연결은 연구 판정 변경이 아니므로 이번 뷰어 작업에서 TeX/PDF를 추가 수정하지 않았다.
- 변경 전 source/PDF/bundle과 당시 작업본을 [로컬 복구 manifest](../.latex-build/cg_dev_entry_before_20260928/manifest.json)의
  `before_change` hash로 보존했다. 기존 사용자 TeX 초안3개는 수정하지 않는다.
- Sketch/master/R1/R2 XeLaTeX 빌드 성공(PDF 각각31/7/40/11쪽), 경고·미해결 참조 없음.
  두 bundle의3/20항목 무결성·변경 대상 일치와 나머지 항목 보존, 문서 링크148개 및 완료 체크 보존을 확인했다.
  GPU 실행·학습·commit/push·외부 fetch/다운로드는 하지 않았다.

## 이전 v12 완료 분석 동기화


- 2026-09-28: v12 GTX1080Ti 세 씬의 연속 완주와 사각형 조건부 복구를 R1의
  [접촉 개발 근거](../development/r1_teacher_probe_oracle.tex) `sec:r1-v12-completion`에 반영했다.
  [결과·원본·실패 분모와 사후 검사 범위](../../experiments/R1_teacher_velocity_reset/self_contact/three_scenes_gpu_v12.md#v12-사용자-실행-결과).
- 저장 원본의 무결성·검산 기록·연속 경계를 확인했으며 새 물리 잔차/CCD 재계산은 하지 않았다.
  GTX1080Ti 완주는 준비 상태를 대체하지만 기본 dt 안정성, RTX5070, 수렴·GS/oracle·학습 적격성은
  미완료다. 기존 R1 완료 체크와 방법 claim은 유지한다.
- 다음은 v12 시각 확인과 수렴 검증 조건 검토다. 추가 GPU 실행 승인은 아니다.
- R1 XeLaTeX/latexmk 빌드 성공(PDF39쪽·521,597 bytes), 경고·미해결 참조 없음.
  Master bundle20항목 중 R1 TeX/PDF2개를 동기화하고 나머지18개의 byte·ZIP 무결성을 확인했다.
  기존 사용자 TeX 초안3개를 보존했다. 구현·실험 원본 수정과 commit/push는 하지 않았다.

## 이전 v12 준비 문서 동기화


- 2026-09-27 작업본 정리: R1 GPU 접촉 절에 cuDSS 설정의 장치 내부/장치 간 한계,
  v11 실패·국소 code1 복구와 v12 연속10초 실행 준비를 반영했다. 새 전체 궤적은 미검증이며
  canonical 체크·Gate·방법 claim은 유지한다. [실행 계약과 검증](../../experiments/R1_teacher_velocity_reset/self_contact/three_scenes_gpu_v12.md#검증과-한계).
- R1 XeLaTeX/latexmk 빌드 성공(PDF38쪽, 518,681 bytes). 경고·미해결 참조 없음.
  Master bundle20항목의 무결성과 working file 일치를 확인했고 이번에는 R1 TeX/PDF만 갱신했다.
- 관련 구현 회귀33개 통과. 본 GPU 시뮬레이션은 실행하지 않았다. 코드·실험·R1 작업본을
  저장소별 commit/push 대상으로 정리하며, 기존 사용자 TeX 초안3개는 제외·보존한다.
- 대응 구현은 code `a5e4d8e`, 실행 계약·진단 기록은 experiments `41ac8f3`에 보존했다.
  동결 run의 실제 소스 식별은 각 manifest의 runtime hash가 우선이다.

## 이전 v10 문서 동기화


- 확인일: 2026-09-22. R1 접촉 절에 실제 실패 code2 재현·cycles6 미해결·GPU dt절반 자동 복구 통과와54회귀를 반영했다. 해당 프레임 해결과 세 씬 장기/5070/시간 수렴 미완료를 구분한다.
- 승인 prefix·유한 통계 확인, GPU 분기/외력 재사용/검산/rollback과 실패 비용 포함을 명시했다. 과거 v8 외부 종료 추정은 정정했고 v9은 당시 구현으로 보존한다.
- 상세 근거는 [복구 승인·검증 근거](../../experiments/R1_teacher_velocity_reset/self_contact/frame112_recovery.md#복구-승인과-검증). R1 체크·Gate·학습 적격성은 변경하지 않았으며 sketch/master source 변경은 불필요하다.
- 문서 재감사에서 final code/experiment commit 연결 누락과 2026-09-14 절의 현재형 ‘접촉 모델 미구현’ 문장을 확인했다. R1에 versioned commit f6db052/a2c42c1, dirty run hash 우선 원칙, 당시 상태와 후속 v10의 구분을 반영했다.
- Canonical sketch의 self-contact 비주장과 target runtime 범위는 유지한다. Master의 R1 진행 상태·Gate·dependency도 바뀌지 않아 두 source는 수정하지 않았다.
- R1 XeLaTeX/latexmk 재빌드 성공(PDF38쪽·516,220 bytes·관련 경고/미해결 참조 없음). 지정 master bundle20항목 중 R1 TeX/PDF와 이전 구조화에서 바뀐 development README 3개를 working file과 일치시켰고, 나머지17개 byte와 ZIP 무결성을 확인했다.
- 당시 기존 사용자 untracked TeX3개는 수정하지 않았다. 당시 미커밋 갱신도 이번 정리 범위에 포함한다.

- 상세 위치: [R1 구현·접촉 계약](../development/r1_teacher_probe_oracle.tex)의 `sec:r1-implementation`, `sec:r1-gpu-self-contact`; [파트별 본문 갱신 위치](../development/README.md#r1-문서-안에서-구현-근거를-갱신하는-방법).

## 이전 구현과 검증 — 당시 상태

- 확인일: 2026-09-22. R1에 GPU 접촉 전 단계·정밀 기하·BVH v5, v9 자동 시간 보정·solver code 보존·제한적 code2 복구를 반영. Canonical acceptance는 미완료다.
- V5 code99는 GMRES720 한도 solver 오류를 audit가 덮은 것으로 분리했다. v9은 원래 code2만 frame-start hi/lo에서 cycles3→6으로 한 번 GPU 재실행하며 다른 오류를 숨기지 않는다.
- GTX1080Ti/RTX5070에서 달랐던 raw PTX 환산은 graph 전체 marker와 host wall의 프레임별 배율로 보정하고 raw를 보존한다.
- GPU 회귀113개 통과·v9 세 씬 bundle 준비까지만 확인했다. 과거 실패 frame 복구와 장기 완주·학습 적격성은 미완료다.
- v5 직사각형은 preload120·calm240을 완료했지만 wind105번째 시도에서 검산 실패·GPU rollback으로 종료됐다. 장기 완주 근거로 승격하지 않고 실패 분모를 보존했다.
- v8은 수치 기준을 바꾸지 않고 프레임별 solver/collision/audit 시간을 기본 출력한다. 관련 GPU 회귀140개와 세 씬384단계 smoke를 통과했고 본 실행은 대기한다.
- BVH 순회/후보별 거리 분리와 임시 raw 초과 시 기존 GPU 전체 재탐색을 기록했다. 후보 잘라내기·승인 조건 변경과 구분한다.
- 기존139개 회귀·동결v5 국소/세 씬 검증 및 동일 입력 제한 프레임 비교와 후속 v8 140개 회귀를 반영했다. 장기 접힘과 변동·추가 버퍼 비용의 한계를 유지한다.
- 공통 항 재사용·GPU CCD 큐·제한 worker의 검증과 추가 버퍼/launch 비용을 반영했다. 공유 GPU 결과의 가속 채택은 보류한다.
- 실제 동결v1 대비 제한 프레임 개선과 같은 실행기의 ON/OFF 비용을 기록했다. 마지막 병렬화만의 일관된 추가 가속은 미확인이며 장기 비용으로 외삽하지 않는다.
- 재사용·작업 배열 초기화·빈 접촉 분기·검산 Hessian 제거는 승인 조건 변경과 구분했다. 공유 GPU 측정은 최종 가속 근거로 채택하지 않는다.
- GPU LBVH refit/미분/adjoint/CCD/솔버·독립 검산·프레임 복원과 CPU oracle 대조를 개발 후보로 기록했다.
- FP64 hi/lo와 실제 FP64 접촉, 보수 FP32 AABB를 구분했다. 기존 contact-OFF M1/M2/Gauss 선택은 유지한다.
- 초기 v1 기하 거절은 보존하고, v2 Gram 행렬 Bernstein·선택적 영역 세분화로 기존12구간을 인증한 근거를 반영했다.
- 비퇴화 요구·물성·barrier·dt를 유지했다. 국소6사례56단계 전부 승인과 실제 퇴화·한도 초과 거절 등106개 검사를 구분했다.
- 새 세 씬 GPU 개별 스크립트·384단계 검산은 반영하되 실제 장기 접촉 완주로 승격하지 않았다.
- Proxy 곡면 전역 인증·응답 공간 수렴·마찰·실장면 calibration·RTX5070·R1/학습 적격성은 남아 있다.
- R1 장/단계 체크와 Gate·contact-off 연구 계약은 올리거나 해제하지 않았다. Sketch/master의 방법·claim 변경은 불필요하다.
- R1 XeLaTeX/latexmk 재빌드 성공, nonempty PDF와 최종 로그의 경고·미해결 참조 없음을 확인했다. Master bundle의 R1 TeX/PDF를 갱신하고 ZIP 무결성·원본 일치를 검증했다.

수치·명령·source/hash: [v2 실험](../../experiments/R1_teacher_velocity_reset/self_contact/refined_geometry.md).
후속113개 회귀·최종v3 GPU 검증·재측정: [성능 점검](../../experiments/R1_teacher_velocity_reset/self_contact/performance.md).
현재121개 회귀·v4 제한 GPU 검증: [병렬 개선](../../experiments/R1_teacher_velocity_reset/self_contact/parallel_v4.md).
새 성능·비용의 조건·원본·분모: [유휴 재측정](../../experiments/R1_teacher_velocity_reset/self_contact/idle_performance.md).
추가 BVH 개선·제한 가속·현행 실행: [v5 보고](../../experiments/R1_teacher_velocity_reset/self_contact/broadphase_v5.md).
기본 진행 계측·장기 실패: [v8 보고](../../experiments/R1_teacher_velocity_reset/self_contact/frame_timing_v8.md).
자동 시간 보정·code2 복구: [v9 보고](../../experiments/R1_teacher_velocity_reset/self_contact/frame_timing_v9.md).
구현: [code 기록](../../code/sessions/2026-09-22_01_self_contact.md).
기존 사용자 untracked TeX3개는 보존했다. 원격 fetch·commit·push는 하지 않았다.

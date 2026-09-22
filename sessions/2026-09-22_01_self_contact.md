# CPU/GPU 셀프 접촉 후보의 R1 상태 반영

## 현재 상태

- 확인일: 2026-09-22. R1 접촉 절에 실제 실패 code2 재현·cycles6 미해결·GPU dt절반 자동 복구 통과와54회귀를 반영했다. 해당 프레임 해결과 세 씬 장기/5070/시간 수렴 미완료를 구분한다.
- 승인 prefix·유한 통계 확인, GPU 분기/외력 재사용/검산/rollback과 실패 비용 포함을 명시했다. 과거 v8 외부 종료 추정은 정정했고 v9은 당시 구현으로 보존한다.
- 상세 근거는 [복구 승인·검증 근거](../../experiments/R1_teacher_velocity_reset/self_contact/frame112_recovery.md#복구-승인과-검증). R1 체크·Gate·학습 적격성은 변경하지 않았으며 sketch/master source 변경은 불필요하다.
- R1 XeLaTeX/latexmk 빌드 성공(PDF38쪽·nonempty·최종 경고 없음), 지정 master bundle20항목의 CRC와 R1 source/PDF byte 일치를 확인했다. 기존 사용자 TeX3개는 수정하지 않았고 푸시 전 원격 동기(HEAD...origin/main `0/0`)를 확인했다.

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

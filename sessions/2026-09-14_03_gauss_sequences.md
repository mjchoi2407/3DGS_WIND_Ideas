# R1 Gauss 세 씬 실행 계약 반영

## 현재 상태

- 확인일2026-09-14. 사용자 선택의6차8분할 샘플을 세 씬의 중력/무풍/바람 분기에 연결한 범위와 기록 계약을 R1에 반영했다.
- 전체 적분 단계의 GPU 검산과60Hz raw 상태 기록을 구분한다. 모든stage teacher 원본 발행이나 전체 궤적/학습 정확도 완료로 승격하지 않는다.
- 설정·검증·run hash는 [실험 보고서](../../experiments/R1_teacher_velocity_reset/timestep_search/cloth_coarse/gauss6_bend500_three_scenes.md), 구현은 [code 계약](../../code/docs/gpu_gauss_solver.md)이 소유한다.
- R0/master/sketch의 claim·완료 기준과 기본 Newmark/Gate는 변경하지 않았다. R1 XeLaTeX PDF34쪽 빌드 통과, 전달 bundle은 R1 TeX/PDF만 갱신하고 나머지 항목 보존을 확인한다.
- 본 실행/서브컴 GPU 실행·외부 fetch·stage·commit·push 없음.

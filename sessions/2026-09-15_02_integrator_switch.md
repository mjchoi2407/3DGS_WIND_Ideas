# R1 Newmark/Gauss 전환 개발 근거

## 현재 상태

- **후속 기하 전용 시험 완료:** 같은 두 국소 입력의4회 선택 모두 Newmark 단일 계산/검산 통과.128분할 감시 없음. 프리로드1~2초 범위 재확인; 시간 정확도 개선은 아님. 실제 기하 경고 해결률/장기 teacher 채택은 미완료. [설정·수치·검증](../../experiments/R1_teacher_velocity_reset/timestep_search/cloth_coarse/geometry_switch_trial.md).

- 후속 사용자 확인: 추가128분할 시간 오차 감시를 제외하고 기하 flag16 단독 때만 재계산하는 별도 `geometry` 시험으로 전환했다. 다른 물리/솔버 실패는 그대로 실패하며 기본 생성기/teacher 기준은 유지한다.

- 확인 기준: 2026-09-15. R1에 FP64 적분기 선택, 같은 원본에서의 재계산, 적분기별 독립 검산과 시간 지표의 개발 범위를 반영했다.
- 두 국소 상태는 원래 검산을 통과했지만 모두 Gauss로 재계산해 기본 가속 채택 근거가 되지 못했다. 작은 외력 변경/연속 전환 fixture와 전체 생성기 검증을 구분한다.
- 전환 지표를 공식 teacher 허용오차로 승격하지 않았다. R1 canonical 체크/Gate·R0 계약·master·sketch는 변경하지 않았다. 장기 시간/공간 수렴·GS/oracle·접촉/학습 적격성은 미완료다.
- [실험 소유 문서](../../experiments/R1_teacher_velocity_reset/timestep_search/cloth_coarse/integrator_switch_trial.md)에 비용 편차·실패 분모·참조 차이와 임계값 민감도를 연결했다.
- R1 canonical XeLaTeX 빌드 및 non-empty PDF, master delivery bundle의 R1 source/PDF 갱신과 나머지18항목/CRC 보존을 확인했다.
- ideas worktree에 변경을 남겼으며 stage·커밋·푸시 없음.

- 기하 전용 정책을 R1 본문에 후속 시험으로 반영했다. XeLaTeX 빌드 성공, R1 PDF와 지정 bundle 갱신/CRC 검증 완료. R0/master/sketch 및 Gate 변경 없음.

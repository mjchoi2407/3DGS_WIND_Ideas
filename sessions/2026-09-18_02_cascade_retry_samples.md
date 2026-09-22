# Newmark half2 후 Gauss8 고부하 표본 비교

## 현재 상태

- 확인 기준: 2026-09-18. 제한 비교 완료, 기존 기본 정책 유지.
- R1에 다단계 복구의 승인 상태 복원·검산 경계와 국소 결과 링크를 추가했다. R1 PDF 빌드 및 bundle 갱신을 완료했다. R0 비승계 경계를 확인했으며 master/sketch/Gate 변경은 없다.
- 세 표본×두 정책×두 반복의 기존 검산 통과. 일부 실패 표본의 시간은 감소했지만 끝 속도 차이가 크므로 동일 정확도의 teacher 가속으로 채택하지 않는다.
- 재시도 없는 고비용 표본에는 개선 근거가 없다. 실제 half 실패→Gauss 고하중 회복은 미관측이며 작은 GPU 실패 주입 검사만 통과했다.
- 정확도·적격성 예산 미정, 장기 궤적/손수건/5070은 미검증. 기존 M1/M2/R64·생산/학습 상태·기존 Gauss와 half 스크립트를 유지한다.
- [원시 근거·측정 한계·재현 및 판정](../../experiments/R1_teacher_velocity_reset/timestep_search/cascade_retry/report.md).
- 다음: 승인된 정확도 예산이나 별도 참조 검증 없이 기본값으로 승격하지 않는다.
- 관련 worktree에만 변경을 남겼으며 stage/commit/push 없음.

# Frame225 비선형 승인 범위 반영

## 현재 상태

최종 결정: frame225 추가 최적화 분기 종료. [종료 문서](../../experiments/R1_teacher_velocity_reset/timestep_search/precision_v3/fresh_branch_closeout.md)에 다섯 판정 층을 분리했다.
Frozen linear 통과/국소 가속, λ=1/8 승인과 잔차 감소율을 보존하며 전체 Newton/substep/audit는 미검증, 운영 채택은 보류다.
수치 실패·전체 성능 저하 확정이나 전체 teacher 가속으로 해석하지 않는다. 추가 계산 없이 M1/M2·HL01 R64·summary/full을 유지한다.
5070 Graph 장애와 학습 적격성/정확도 예산은 별도 미해결이다. 원본 artifact와 ZIP은 보존했다.

확인 기준: 2026-09-17. R1에 저장 보정의 line search 확인과 그 한계를 반영했다.

- [R1](../development/r1_teacher_probe_oracle.tex): fresh의 선형 잔차/가속과 비선형 잔차 감소를 분리했다. 전량 보정 거부와 축소 보정 승인만 확인했으며 timestep 완료가 아니다.
- 후속 상태는 line_search_checked_frozen_candidate다. 기존 HL01 R64 운영·M1/M2 선택과 연구 Gate/생산/학습 적격성을 유지한다.
- 완전한 substep 재시작 계약 미확인으로 Newton/audit 확장은 미실행이다. 새 장시간 재생·frame236/전체 FP32 개발 없음.
- [실측 근거](../../experiments/R1_teacher_velocity_reset/timestep_search/precision_v3/frozen_line_search_report.md)가 상세 수치·hash/원본을 소유한다.
- R1 PDF 실제 XeLaTeX 빌드와 master bundle R1 source/PDF 갱신. R0·master·sketch 계약 변경 없음.
- 이전 closeout 결과는 보존했다. 커밋·push 없음.

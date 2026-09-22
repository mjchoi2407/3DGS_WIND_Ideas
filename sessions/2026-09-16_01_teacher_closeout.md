# Teacher 가속 개발 종료 경계 반영

## 현재 상태

확인 기준: 2026-09-16. 사례별 가속 후보 동결과 두 제한 시험의 완료 근거를 R1에 반영했다.

- [R1](../development/r1_teacher_probe_oracle.tex)의 구현 상태와 fixture 절에 M1/M2/R64 역할, 환경 한계와 제한 종료를 반영했다.
- W30 기록 모드 비교는 기존 검산·저장 보존을 통과했으나 비영 수치 차이가 있어 회귀 동등성은 미판정이다.
- HL01 frame225 F64_fresh의 국소 기준/비용 통과를 promising_frozen_candidate로 기록했다. frame236/전체 고하중/전역 P 갱신으로 확대하지 않는다.
- 5070 성공 API13000과 현재 API13040, Graph 미해결을 분리한다. 완료 W30 2쌍만 집계하며 겹치는 구간 합산을 금지한다.
- R1 canonical PDF를 XeLaTeX로 실제 빌드하고 master 전달 bundle의 R1 source/PDF를 동기화했다.
- R0 하위 구현과 전체 계약 경계를 확인했다. 이번 변경은 target schema/계약을 바꾸지 않아 R0 수정·재빌드는 불필요하다.
- R1 공간·시간 수렴/장기 적격성/GS·oracle, R0 전체 계약과 연구 Gate는 미완료다. Master/sketch 및 완료 체크는 변경하지 않았다.
- 상세 결과·hash·명령은 [실험 종료 보고서](../../experiments/R1_teacher_velocity_reset/timestep_search/precision_v3/closeout_report.md)가 소유한다.
- 첨부 보고서/closeout 문서는 미발견하여 사용자 메시지 범위로 판단했다. 첨부 대조는 미완료다.
- 기존 사용자 변경을 보존했으며 커밋·push하지 않았다.

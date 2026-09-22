# R1 GPU Gauss 개발 근거 반영

## 현재 상태

- 후속 타임스텝 실험의 국소 비용 감소 근거와 정확도 분리 원칙을 R1에 추가했다. [타임스텝 보고서](../../experiments/R1_teacher_velocity_reset/timestep_search/cloth_coarse/gauss_timestep_sweep.md)를 연결하며 기본값·Gate·R0/master/sketch 판정은 유지한다.
- 확인일2026-09-14. R1에3단계6차 Gauss의 GPU 적분/보조 풀이/독립 검산과 기존 Newmark 대비 국소 비교의 범위를 반영했다.
- 실제 단계별 힘/HVP를 유지하고 공통 접선은 보조 풀이에만 쓴다. GPU 검산은 CPU longdouble 식 및 P3×cubic 구간 기하와 대조했다.
- 비용/오차/실패 분모/원본은 [실험 보고서](../../experiments/R1_teacher_velocity_reset/timestep_search/cloth_coarse/gauss_gpu_comparison.md), 구현 계약은 [code 문서](../../code/docs/gpu_gauss_solver.md)가 소유한다.
- 기본 Newmark/FP64 hi/lo·물성·학습 label·연구 Gate는 변경하지 않았다. 전체 시간/공간 수렴·GS 매핑·접촉·학습 적격성은 미완료다.
- R0/master/sketch의 claim/단계 판정은 변경하지 않았다. R1 source/PDF와 전달 bundle만 이번 갱신 범위다.
- `latexmk -cd -g -xelatex -interaction=nonstopmode -halt-on-error development/r1_teacher_probe_oracle.tex` 빌드 성공, non-empty34쪽 PDF 확인. 전달 bundle의 R1 TeX/PDF2항목만 교체하고 전체20항목 CRC와 나머지 byte 보존을 검증했다.
- 외부 fetch·stage·commit·push 없음. 기존 dirty 변경은 보존했다.

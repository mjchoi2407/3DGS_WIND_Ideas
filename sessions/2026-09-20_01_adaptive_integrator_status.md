# R1 GPU 적분기 선택 상태 동기화

## 현재 상태

- 확인 기준: 2026-09-20.
- R1에 GPU별 Newmark M1/M2, 직접 Gauss R64/mixed32와 Newmark→Gauss 전환 세 조건을 제한된 개발 후보로 반영했다.
- 분기는 프레임 시작 FP64 hi/lo 복원, 실패 비용 포함, 기존 정확도·독립 검산 유지를 계약으로 한다.
- 세 장면 장기 결과, teacher 적격성, 5070의 driver API13000 외 Graph 환경은 미검증이다.
- R1 장/단계 체크와 Gate는 올리지 않았다. Sketch·master의 방법/claim은 변경되지 않았다.

# 기여와 다밀도 GS 학습 목적 명확화

## 현재 상태

- 확인 기준: 2026-09-13. 사용자 논의를 현행 스케치의 설명에 반영하고 PDF·전달 bundle 갱신을 완료했다. 물리·학습 실험은 실행하지 않았다.
- 근거 문서: [스케치 TeX](../3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex), [PDF](../3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.pdf)의 「검증 대상 기여」·「Independent multi-resplat pairing」·최소 baseline 목록.
- Teacher의 공간·시간 수렴은 데이터 품질의 전제이며 자체 솔버 구현을 주요 AI 기여로 세지 않는다. Noels (2009)와 현행 P3의 관계는 참고한 연결 원리로 한정하고 원 논문 재현·개선·안정성 보장 승계를 주장하지 않는다.
- MC1은 GS 표현 차이에 대한 면적·질량·바람의 일·응답 일관성, MC2는 새 static GS에서 target mesh·대상별 fitting/재학습 없는 response distillation, MC3는 바람에 따른 선택적 Global/Local 실행의 정확도–시간 이득으로 명확히 했다. SYS는 이를 완성하는 시스템 항목이다.
- 물체별 검증 후 선택한 하나의 teacher 해상도에서 생성한 동일 응답을 여러 독립 GS에 mapping한다. 모든 물체에 같은 vertex 수를 강제하지 않으며 별도 mesh/time refinement는 유지한다.
- GS 생성 시 Gaussian 예산·densification 제한을 달리하는 독립 최적화와 원본 random subsampling을 구분했다. 밀도 제한 비교는 공통 이미지·카메라와 가능한 공통 seed/최적화 조건을 사용하고, seed/view 변화는 별도 축으로 다룬다.
- 단일 GS·random subsampling·독립 다밀도 GS 학습을 공통 teacher·분할·모델 규모·학습 budget에서 비교하고, 다밀도 조건 안에서 measure/consistency의 추가 효과를 분리하도록 명시했다. 같은 물체의 재표현 일관성과 미관측 물체 일반화도 구분한다.
- 기존 수식·정확도 기준·Gate·완료 판정은 변경하지 않았다. 다음 검증은 기존 R1/R3 계약에 따른 teacher 적격성 및 독립 multi-resplat 학습 효과 확인이다.

## 검증 및 변경 범위

- `ideas/`에서 `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error 3dgs_response_distilled_global_local_wind_dynamics_2026-08-22.tex` 실행 성공. BibTeX 및 후속 pass를 포함하여 31쪽 PDF를 생성했다.
- 최종 LaTeX 로그의 오류·미해결 참조·레이아웃 경고가 없음을 확인하고, 수정 핵심 부분인 PDF 4·5·11쪽을 렌더링하여 확인했다.
- 스케치 bundle의 기존 구성인 TeX·PDF·Bib 3개를 유지하고 ZIP 무결성 및 각 항목과 현재 원본의 바이트 일치를 확인했다.
- 이번 변경은 스케치 TeX/PDF/bundle과 이 기록·session 진입점에 한정했다. 기존 dirty 변경을 보존했으며 stage·commit·push·fetch는 수행하지 않았다. 로컬 현행 문서와 사용자 논의를 기준으로 정리했으며 새로운 외부 문헌 조사는 수행하지 않았다.

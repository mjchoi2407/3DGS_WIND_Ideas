# TD00 구현 계약에 따른 방법 문서 보정

## 목적

새 아이디어 스케치와 구현 체크리스트가 TD00 code contract와 같은 Global--Local force 흐름을 설명하도록 맞춘다.

## 보정 내용

- Module 7을 network-only 단계가 아니라 active patch의 세 force proposal을 묶는 orchestration 단계로 명확히 했다.
  - Module 3 analytic law의 Local base aero
  - learned missing aero
  - learned missing structural
- 세 channel 모두 overlap assembly(Module 10), Global complement(Module 9), 단일 Local physical solve(Module 8)를 거치도록 명시했다.
- Module 5가 소비한 이전 Local cross load를 기록하고, Module 11은 updated coupling과의 차이인 `structural_cross_delta`만 소비하도록 수정했다.
- RSH는 mainline이 아니라 TD06/E2의 opt-in representation ablation으로 유지했다.
- 두 TeX를 XeLaTeX로 다시 빌드하고 PDF와 전달 bundle을 갱신했다.

## 검증

- 아이디어 스케치 64쪽, 구현 체크리스트 33쪽 PDF 빌드가 완료됐다.
- undefined citation/reference/control-sequence가 최종 log에 남지 않았다.


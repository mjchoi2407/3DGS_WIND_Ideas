# Wind3DGS Ideas Instructions

연구 명세 저장소다. [공통 필수 지침](../AGENTS.md)을 적용하며, 하위 저장소 단독 실행에서 상위 지침이 제공되지 않았다면 먼저 확인한다. 이미 확인한 지침은 재출력하지 않는다.

## Startup Protocol

- 맥락이 충분하면 진행한다. 부족할 때만 [주제별 작업 색인](sessions/README.md#현재-상태) → 해당 note의 현재 상태 → 상세 절 링크 순서로 확인한다.
- 현행 방법·claim은 [canonical sketch 진입점](README.md#현재-아이디어-스케치), 단계·구현 계약은 [R0–R7 경로](development/README.md#master-roadmap)의 해당 파트만 읽는다. 최신 파일명이나 과거 note로 현행 방향을 추정하지 않는다.
- Framing/pivot은 sketch 관련 절, 구현·milestone은 master와 해당 R 절, novelty는 필요한 관련연구·참고문헌만 읽는다.
- 외부 리뷰 대응에만 [리뷰 절차](review_workflow.md)를 적용한다. 연구 정리·TeX·전환 작업은 공통 지침의 해당 조건부 절차를 따른다.

## Editing Rules

- 사용자 파일과 기존 근거를 보존한다. 구현 상세는 pseudocode·설계·실험 요구 외에는 code에 둔다.
- 연구 결과·계약 동기화는 [R1 갱신 위치 안내](development/README.md#r1-문서-안에서-구현-근거를-갱신하는-방법)와 해당 R의 종료 조건을 따른다. 부분 검증을 완료로 승격하지 않는다.
- 방향 전환은 [보존·전환·재검증 순서](../docs/workflows/research_and_latex.md#연구-방향-전환-절차), TeX 수정은 [canonical PDF·bundle 빌드](../docs/workflows/research_and_latex.md#latex-pdf-빌드-규칙)를 적용한다. 기존 사용자 TeX 변경을 임의로 인수·빌드하지 않는다.

## Canonical 산출물

현재 [산출물 정책](README.md#canonical-산출물-정책)과 `.gitignore`의 exact allowlist를 따른다. Canonical 경로·전달물 변경 시 [정본 전환과 산출물 규칙](../docs/workflows/research_and_latex.md#ideas-정본-전환과-산출물)을 먼저 확인한다. Archive·임시 preview는 현행 근거가 아니며 반복 생성·배포하지 않는다.

## Session Tracking

확정 결정·검증 범위·남은 조건은 [공통 기록 규칙](../AGENTS.md#간결한-작업-기록)에 따라 해당 note에 갱신한다. [주제별 색인](sessions/README.md#현재-상태)은 연구 상태와 상세 절 링크만 두며 실행 수치·원본은 experiments를 연결한다.

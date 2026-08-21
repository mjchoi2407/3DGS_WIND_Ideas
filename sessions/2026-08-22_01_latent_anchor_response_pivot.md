# 2026-08-22 01 latent anchor response pivot

## Context

기존 Minimal V0는 static 3DGS에서 explicit anchor/edge/triplet scaffold를 증류하고 structural/reduced
package를 구성하는 방향이었다. 개념 가이드를 검토하면서 target inference에 simple simulation mesh와
유사한 persistent connectivity를 만드는 것이 연구 질문과 구현 scope를 복잡하게 한다는 점을 재검토했다.

LaGSplat, NeuROK, Simplicits와 FreeForm을 비교한 뒤, mesh connectivity는 training-only teacher로 활용하되
target에서는 static GS로부터 force-to-motion response를 직접 예측하는 방향을 선택했다. Universal model보다
지원 domain을 좁혀 champion scene과 작은 object-disjoint suite에서 성공 가능성과 novelty를 함께 검증한다.

## Decisions

- 새 working title은 *Response-Distilled Global--Local Wind Dynamics for Static 3D Gaussian Thin Surfaces*다.
- 정확한 claim은 `fully meshless`가 아니라 `target-mesh-free inference without a persistent physical adjacency graph`다.
- 입력은 rest static 3DGS, metric scale, known material, hard attachment와 prescribed wind다.
- Mesh vertex/connectivity, surface quadrature, simulation trajectory와 mesh--GS correspondence는 training-only다.
- 모든 mesh vertex는 dense teacher sample로 사용하지만 vertex 1개를 latent token 1개와 일대일 대응하지 않는다.
- Encoder는 patch-local aggregation 뒤 global token context를 사용한다. Latent anchor/relation은 내부 표현이며 외부 graph가 아니다.
- Setup network output은 absolute deformation이 아니라 area/mass measure, passive Global field/poles/damping과
  complementary Local response dictionary를 포함한 cached response package다.
- Runtime frame evaluator는 wind load를 package에 투영해 response increment를 계산하고 deterministic하게 state를 갱신한다.
- Global은 항상 활성이고 Local은 frozen Global residual만 학습한다. Local은 Global/measure Gate 통과 뒤에만 구현한다.
- Exact attachment, positive pole/damping과 mass whitening은 learned penalty가 아니라 deterministic construction이다.
- 핵심 novelty 후보는 GS measure/density consistency, topology-privileged target-mesh-free response distillation,
  wind-residual-driven selective Global--Local execution이다. Transformer/latent token/virtual-work 자체는 기여로 주장하지 않는다.
- Core domain은 flat-rest attached strip/rectangular flag, known 2--3 material presets, prescribed one-way wind와
  small-to-moderate deformation이다. Severe fold/self-contact/tearing/wake/unknown material은 제외한다.
- Champion scene은 사후 cherry-pick이 아니라 dev에서 동결한 지원 domain 안에서 선택하고, object-disjoint quantitative suite와 failure case를 함께 둔다.
- Venue 전략은 SCA/CGF를 현실적 첫 목표로 두고, evidence가 강해질 때 Eurographics/TVCG/SIGGRAPH/TOG로 올린다.

## Changed Files

- 새 canonical sketch, bibliography, implementation checklist를 2026-08-22 파일명으로 작성했다.
- `README.md`와 `.gitignore`의 canonical pointer/allowlist를 새 방향으로 전환했다.
- 이전 canonical sketch/checklist/Bib/PDF/bundle 및 concept guide 9개를
  `backup/minimal_v0_before_latent_anchor_response_2026-08-22/`로 이동하고 checksum을 보존했다.
- Review archive의 당시 검토 대상 링크를 새 sibling archive 위치로 갱신했다.
- 새 PDF 빌드 중간산출물은 top-level에 남기지 않았다. 최초 draft, contract-closure 감사본과 최종
  release build를 `backup/local_workspace_cleanup_2026-08-21/build_artifacts/` 아래 구분된 세 디렉터리에
  로컬 전용으로 보존하고 전체 checksum manifest를 갱신했다.

## Compatibility Audit

`../code`와 `../experiments`는 읽기 전용으로 감사했다. 기존 `TD##`, topology/scaffold, fixed-Hessian PD,
generalized-eigen Global/patch Local, Active/Decay 경로는 새 방법의 구현 완료 증거가 아니다.
Static GS loader, mesh teacher, renderer/transport fixture는 R0 contract를 통과한 뒤 선택적으로 재사용할 수 있다.
이번 pivot 작업에서는 sibling repository를 수정하지 않았다.

## Verification

두 TeX를 `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 실제 빌드했다.
Sketch PDF는 A4 21쪽, checklist PDF는 A4 9쪽이며 source보다 최신이고 non-empty다. 최종 log에는
LaTeX/package error, undefined reference/citation, rerun, multiply-defined label, overfull/underfull과
font substitution이 없다. Ghostscript 전 페이지 decode와 PDF text extraction, 표지·I/O·Global/Local·Gate·DoD
대표 페이지 raster inspection을 통과했다.

후속 PDF 확인에서 `xeCJK` 기본값이 한글 문자 사이의 원문 공백을 제거하는 문제를 발견했다. 두 canonical
TeX에 `\xeCJKsetup{CJKspace=true}`를 추가하고 다시 빌드했다. TeX 원문, PDF text layer와 raster preview에서
`이 문서는`, `현재 연구 질문`, `한눈에 보는 아이디어` 등의 띄어쓰기가 일치함을 확인했다. 페이지 수는
sketch 21쪽, checklist 9쪽으로 유지됐고 최종 log의 오류·경고·overflow 검사는 다시 통과했다.

Sketch bundle은 TeX/Bib/PDF 3개, checklist bundle은 TeX/PDF 2개만 포함하며 `unzip -t`와
bundle 내부--working file SHA-256 동일성 검사를 통과했다. 최종 hash는 다음과 같다.

- sketch TeX: `f9c382959e22d798bb80476ec6277c7f0119ca0058017272b80cef5abeba479c`
- sketch Bib: `9b92c3ff84e2fd9ffde364728b2552bd89c67bdf8e383f831a630a86de1b3923`
- sketch PDF: `803f36e30e54af86a20f9249dc9a49391821f41451fcbb4892d1dae149940d8a`
- sketch bundle: `0c684cf9e9f72d61959bfff5befe63ce824b27b17b13f03bd3d4599ac51f4485`
- checklist TeX: `69e97c0904f1463f6332c583bdc2a9af9e32484a3fa98665c3c60b30228b0c4a`
- checklist PDF: `391a0c5242ed6cd18eab2b55bc565e98fa1d996549222cf81f929bf815ddc574`
- checklist bundle: `4c728168f437c32e116c7aa56d9610c9de8655884e79b87d61ecf01eef500c46`

이전 canonical archive 9개와 local cleanup manifest 전체를 각각 `sha256sum -c`로 재검증해 모두
일치함을 확인했다. README의 current/archive 링크와 `.gitignore` exact-path allowlist도 존재하는 파일을
가리킨다.

## Next

1. R0 teacher/student visibility와 response-package schema를 code/experiment contract로 옮긴다.
2. R1 strip/flag teacher convergence와 independent GS transport smoke를 만든다.
3. R2 single-case Global overfit 전에는 Local/selector를 구현하지 않는다.
4. Gate A multi-resplat consistency와 Gate B held-out Global이 통과한 뒤에만 Local residual 학습으로 진행한다.

# 외부 지적을 반영한 V0 연구 범위와 수식 계약 보정

## 목적

아이디어 스케치에 대한 외부 검토에서 제기된 네 가지 핵심 수식 문제와 과도한 연구 범위를 반영해,
전체 시스템 구현 전에 검증할 수 있는 V0와 조건부 연구 모듈을 분리했다.
아이디어 스케치와 구현 체크리스트를 같은 owner, state, force, 실험 계약으로 동기화했다.

## V0 범위 재정의

- V0 core를 full-anchor current-state analytic aero, always-on Global ROM,
  patch-local fixed-superset Local physics, fixed-\(K\) Oracle/runtime-residual selector,
  joint conservation--complement force assembly, 한 번의 Local solve와 Global delta corrector로 제한했다.
- Aerodynamic hyper-reduction(H), learned missing force(F), learned future-benefit gate(G)는
  각각 독립적인 profiling/Oracle kill test를 통과할 때만 추가하는 조건부 모듈로 내렸다.
- Material과 attachment는 static 3DGS에서 자동 식별하는 runtime 변수가 아니라
  metric scale과 함께 setup metadata로 고정했다.
- 표현 주장은 `mesh-free` 대신
  `without explicit target triangle-mesh reconstruction`으로 제한했다.
- 첫 논문의 중심 기여를 setup-conditioned scaffold, error-triggered complementary Local dynamics,
  closed-loop geometry-to-load/직접 렌더링 transport의 세 항목으로 정리했다.

## 핵심 수식과 구현 계약 보정

- 모든 patch basis를 전역 QR/SVD로 섞는 구성을 제거했다.
  각 patch에서 Global span만 제거한 뒤 retained Gram eigenspace에서 정규화하고,
  inter-patch \(M/K/C\) cross block과 fixed coordinate ownership을 저장하도록 했다.
- Global predictor가 소비한 이전 Local structural reaction을 저장하고,
  corrector에는 updated-minus-consumed delta만 넣도록 명시했다.
- 3x2 tangent Jacobian의 두 column cross product로 area-normal을 정의하고,
  Gaussian covariance transport용 3x3 full-shell affine map을 별도 typed field로 분리했다.
- Teacher nodal force가 있을 때의 conservative mesh-to-anchor transfer 경로와,
  generalized force만 있을 때의 constrained lifting 경로를 모두 정의했다.
- Force/torque conservation과 \(\Phi^\top f=0\)를 순차 projection하지 않고 하나의 joint KKT로 풀도록 했다.
- Selector activation \(\gamma_r\)는 overlap force assembly에서 정확히 한 번만 적용하고,
  Local \(M/C/K\)에는 곱하지 않도록 했다. Decay patch는 기존 물리 state를 계속 적분한다.
- Runtime physics-residual selector는 Local이 섞이지 않은 동일 Global-only counterfactual state에서
  aero, inertia, damping, internal force와 support mask를 일관되게 평가하도록 수정했다.
- Patch benefit과 GPU cost의 비가산성을 명시하고 V0는 cost model 없는 fixed-\(K\) sweep을 사용한다.

## 연구 순서와 kill test

- Physics Track A0는 Oracle cantilever와 same-anchor sparse PD/XPBD/corotational solver를 먼저 비교한다.
  Dynamics-only 약 2배 이득 또는 동일 latency에서 더 낮은 trajectory/spectrum error가 없으면
  selective-compute quality--latency headline을 철회한다.
- Representation Track B0는 5--10개 thin-surface asset에서 physics track과 병렬로 실행한다.
  Area/mass, 첫 8--16개 eigenfrequency, modal subspace angle과 close-layer false edge를 조기 검사한다.
- Full-anchor aero profiling 뒤에만 H를, physics-only Local gap이 있을 때만 F를,
  Oracle 대비 residual selector gap이 있을 때만 G를 평가한다.

## 수정 파일

- `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.tex`
- `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13.pdf`
- `refs_topology_distilled_error_triggered_wind_dynamics.bib`
- `3dgs_topology_distilled_error_triggered_wind_dynamics_2026-08-13_bundle.zip`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.tex`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13.pdf`
- `implementation_checklist_topology_distilled_global_local_wind_dynamics_2026-08-13_bundle.zip`

기존 workflow policy 관련 변경과 `sessions/2026-08-13_10_workflow_policy_update.md`는
이번 연구 문서 보정과 분리해 보존했다.

## 검증

- 두 TeX를 `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 강제 재빌드했다.
- 아이디어 스케치 73쪽, 구현 체크리스트 39쪽 PDF가 생성됐다.
- LaTeX error, undefined control sequence, undefined citation/reference와 Overfull box가 없다.
- 두 source 모두 `\\end{document}`가 정확히 한 번 있고 뒤에 비공백 내용이 없다.
- 이전에는 문서 끝 뒤에 있어 누락됐던 Runtime 출력과 Training-only input이 최종 PDF 본문에 포함됨을
  `pdftotext`로 확인했다.
- 두 PDF의 첫 페이지를 rasterize해 표지, 요약 상자와 목차 렌더링을 확인했다.
- 스케치 bundle은 TeX/Bib/PDF 세 파일, 체크리스트 bundle은 TeX/PDF 두 파일만 포함한다.
  `unzip -t`가 통과했고 archive 내부 파일과 working file의 SHA-256이 모두 일치했다.
- `git diff --check`를 통과했다.

## 구현 전 남은 결정

- Full-shell normal extension \(\lambda_n\)은 V0에서 1로 고정했으며, 다른 thickness 정책은 후속 결정이다.
- Moving support가 있는 residual counter-state의 reaction 진단 규약을 fixture로 검증해야 한다.
- Strong-overlap patch group과 owner-preserving block pivoting의 구체 알고리즘을 TD03에서 확정해야 한다.
- 조건부 H/F/G의 backbone, tolerance와 profiler threshold는 각 kill test 결과 뒤에만 고정한다.

## 2026-08-14 두 번째 검토 피드백 반영

두 번째 검토 보고서의 권고를 현재 73쪽 canonical sketch와 다시 대조했다.
보고서가 지적한 patch basis, gate label, H/F/G, surface map, Global corrector의
source-of-truth 충돌은 64쪽 이전 snapshot에 해당하며 현재 sketch에는 이미 해소되어 있어
현재 revision의 결함이라는 판정은 수용하지 않았다.
대신 구현 문서가 다시 어긋나지 않도록 sketch를 formula source로 명시하고
equation/tensor/owner/activation 소비 지점의 동기화 검사를 요구했다.

수용한 핵심 변경은 다음과 같다.

- Every-frame full Global-only counter-state residual이 Local skip보다 비쌀 수 있다는 지적을 수용했다.
  V0 selector를 기존 force를 재사용하는 owner-mass-normalized cached dynamic compliance Top-(K)로 바꾸고,
  full counter-state residual은 offline/low-rate analysis ceiling으로 내렸다.
- Strong-overlap patch를 고정 atomic gating unit으로 묶고 Oracle/runtime/learned selector가
  동일 group과 (K_A)를 사용하도록 score, group-to-patch active mapping과 budget 단위를 정의했다.
- Active/decay principal block factorization 위험을 수용했다.
  Same-set exact direct를 numerical oracle, full-superset prefactor를 full-Local upper bound로 분리하고,
  version/integrator/timestep/set signature cache key, block-PCG, decay/set-churn profiler와 P0 kill gate를 추가했다.
- 약 (2\times) speedup과 기타 수치는 출판 합격선이 아니라 개발용 중단 기준임을 명시했다.
  Same scaffold는 same DOF가 아니며 실제 rank/DOF를 별도 보고한다.
- Resolution decoupling은 독립 algorithmic novelty가 아니라 structural dynamics scaling evidence로 제한했다.
- `error-triggered`는 certified a-posteriori estimator가 아니라 Oracle로 검증하는
  omitted-complement response proxy라는 용어 계약을 추가했다.
- Paper/plant의 범위를 permanent creasing 없는 paper-like elastic sheet와
  compliant stem의 single leaf/petal로 좁혔다.
- BendTwin과 FastPhysGS를 related-work 경계에 추가하고 PD/XPBD canonical baseline 인용도 보강했다.

다음 권고는 그대로 수용하지 않았다.

- 독립 prefactor group solve 뒤 Global corrector로 group coupling을 대신하는 처방은 거부했다.
  누락되는 same-step Local--Local effective load는 일반적으로 Global-to-Local coupling operator의 range에 없고,
  Global corrector는 same-frame Local state를 다시 풀지 않기 때문이다.
  Group factor는 cross-block matvec를 보존하는 PCG preconditioner로 쓰며,
  독립 group solve는 coupling/rollout tolerance가 작은 경우의 ablation으로만 허용한다.
- Exact chain이 선행에 없다는 사실만으로 novelty가 성립한다는 해석은 거부했다.
  MC1은 operator/eigenspace 보존, MC2는 강한 Global/same-scaffold baseline 대비
  matched quality--latency 이득으로 각각 입증해야 한다.
- Full-anchor aero와 affine transport 자체는 주기여로 올리지 않고 각각 V0 backend와 system closure로 남겼다.

이번 추가 작업의 직접 수정 범위는 canonical 아이디어 스케치, 해당 Bib/PDF/bundle과 이 session note다.
구현 체크리스트는 사용자 요청 범위가 아니므로 이번 단계에서는 추가 동기화하지 않았으며,
후속 동기화 시 cached-compliance selector와 owner-mass normalization을 반영해야 한다.

최종 XeLaTeX/BibTeX 빌드는 79쪽 PDF를 생성했다.
LaTeX error, undefined control sequence/reference/citation과 Overfull box는 없고,
남은 경고는 CJK italic font substitution과 underfull box뿐이다.
스케치 bundle은 갱신된 TeX/Bib/PDF 세 파일만 포함하며 `unzip -t`를 통과했고,
archive 내부 세 파일의 SHA-256이 working file과 각각 일치한다.

## 2026-08-14 후속 revision review 재반영

후속 보고서 `wind3dgs_revision_review_report_2026-08-14.pdf`를 당시 79쪽 canonical sketch와
대조해 문제 진단과 제안 해법을 분리해서 판정했다. 이번 단계에서는 아이디어 스케치뿐 아니라
구현 체크리스트까지 다시 동기화했다.

### 수용한 내용

- 구현 착수 범위는 전체 예측 시스템이 아니라 TD01 deterministic contract,
  병렬 representation feasibility와 Oracle A0/A1 kill test로 제한한다.
- MC1은 setup-conditioned topology/operator/eigenspace 보존과 downstream rollout의 관계,
  MC2는 strong Global rank, same-anchor solver와 always-on Local 대비 quality--latency Pareto로 입증한다.
- Same-p95 비교를 우선하고 velocity RMSE를 primary accuracy metric으로 사용하며,
  position/trajectory와 spectrum/PSD를 필수 보조 지표로 유지한다.
- Dormant만 완전 skip이며 Decay는 expert와 새 excitation을 끄더라도 물리적 감쇠 solve를 계속한다.
  Stationary/slow/fast/reversing gust별 active--decay union, cache churn과 p95/p99를 별도로 측정한다.
- Runtime joint conservation--complement correction은 generic anchor-primal KKT로 구현하지 않는다.
  Patch copy를 unique anchor support로 먼저 응축하고, low-row constraint inventory에 대한 dual Schur를 풀어
  force를 복원한다. Current-geometry torque row, rank/RHS consistency, factor signature와 fallback을 명시했다.
- Oracle/learned/runtime selector, representation, solver와 render evidence를 서로 대체하지 않도록
  Gate A/B/C와 구현 invariant를 분리하고, coverage hole/overlap/temporal popping도 render 지표에 추가했다.

### 부분 수용 또는 반박한 내용

- `Error-Triggered` 제목은 결과 전 조건부라는 지적은 수용했지만 learned gate G가 제목의 필요조건이라는
  처방은 거부했다. Deterministic cached-compliance도 held-out Oracle rank correlation, regret,
  activation delay/high-benefit miss와 heuristic 대비 quality--latency를 통과하면 제목을 방어할 수 있다.
  실패 시 `Compliance-Guided` 또는 `Selective Complementary`로 낮추며,
  V0 score가 residual이 아니므로 `Residual-Guided`는 사용하지 않는다.
- Selected Local이 always-on Local을 모든 점에서 strict dominance해야 한다는 해석은 채택하지 않았다.
  Matched-quality speedup, same-latency error 또는 multi-object throughput을 포함한 Pareto 비지배 영역으로 판정한다.
  약 2배는 내부 개발 중단 기준이지 출판 보장선이 아니다.
- Coordinate 또는 rank를 억지로 맞추는 비교는 공정성의 primary 조건으로 채택하지 않았다.
  실제 rank/DOF는 공개하되 latency와 품질을 직접 비교한다.
- Gate C 실패가 MC1 또는 Oracle Local accuracy를 자동으로 기각한다는 해석은 거부했다.
  이 경우 selective-compute/SYS headline만 축소하며 Gate A와 Gate B는 독립 판정한다.
- Decay에서 계산을 모두 없애라는 문자적 처방은 물리 state continuity와 충돌하므로 거부했다.
  Actual-skip은 Dormant skip과 Decay의 제한적 비용을 분리해 보고한다.
- `without runtime continuum simulation`은 reduced physics까지 없다는 오독을 피하도록
  `without a runtime full-order continuum solver`로 좁혔다. Interactive/real-time 표현은
  end-to-end p95 측정 전에는 사용하지 않는다.

### 문서와 구현 순서 변경

- 스케치 Module 10에 unique-anchor condensation, weighted projection의 small dual-Schur 식,
  constraint row inventory, infeasibility 처리, geometry-dependent torque와 factor reuse 계약을 추가했다.
- Dense primal solve는 test reference로만 남기고 production path는 equivalent Schur/null-space 또는
  intrinsic antisymmetric force construction만 허용한다. Sequential conservation/Global projection fallback은 금지한다.
- 체크리스트 critical path를 TD07-A local operator, TD11-A joint assembly/complement,
  TD07-B official Oracle Local Gate B, TD10 selector/state machine, TD11-B corrector 순으로 재배치했다.
- P0에 same-p95 accuracy, adaptive overhead, gust/decay accumulation, approximate backend와 KKT implementation
  실패 조건을 추가하고 Gate A/B/C의 기여 범위를 명시했다.
- Working-title evidence gate와 fallback 제목, MC1/MC2 + SYS hierarchy, setup-conditioned/runtime full-order
  경계를 sketch와 checklist에 동일하게 반영했다.

### 검증

- 두 TeX를 `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 다시 빌드했다.
- 최종 PDF는 아이디어 스케치 82쪽, 구현 체크리스트 40쪽이다.
- 최종 log에 LaTeX error, undefined control sequence/reference/citation과 Overfull box가 없다.
  남은 경고는 CJK italic font substitution, hyperref PDF-string 경고와 Underfull box뿐이다.
- PDF text extraction으로 title evidence contract, runtime small-Schur와 TD11-A critical path가 실제 결과물에
  포함됨을 확인했고, 표지·small-Schur 페이지·TD11-A 페이지를 rasterize해 clipping 없이 렌더링됨을 확인했다.
- 두 source의 `\\end{document}`는 각각 정확히 한 번이며 뒤에 비공백 내용이 없고,
  duplicate label과 stale runtime-residual selector 문구가 없음을 확인했다.
- 스케치 bundle은 TeX/Bib/PDF 세 파일, 체크리스트 bundle은 TeX/PDF 두 파일만 포함하도록 갱신했다.
  두 archive의 `unzip -t`가 통과했고 내부 파일과 working file의 SHA-256이 모두 일치한다.

## 2026-08-14 후속 revision review v2 수용·반박 반영

후속 보고서 `wind3dgs_revision_review_report_2026-08-14_v2.pdf`를 당시 82쪽 canonical
아이디어 스케치와 대조했다. 이번 보고서는 최신 revision을 읽었고 새로운 치명적 수식 오류는
제시하지 않았으나, adaptive cost의 용어·계측, patch complement locality, selector blind spot과
실험 범위를 더 엄밀하게 만드는 제안이 있었다. 사용자 요청에 따라 수용한 내용은 본문 계약과
실험 항목에 편입하고, 그대로 채택하지 않은 처방은 관련 장에 보이는 `V2 리뷰 판정` 코멘트로 남겼다.

### 수용한 내용

- `Top-K`의 \(K_A\)는 동일 GPU cost가 아니라 atomic gating-unit 수이므로
  `fixed-budget`을 `fixed-cardinality`로 교정했다. \(K_A\)와 함께 active/decay/solve coordinate rank
  \(r_A,r_D,r_S\), unique support-anchor 수 \(n_A,n_D,n_S\), 실제 p50/p95/p99를 보고하도록 했다.
- Global complement projection이 patch basis에 만드는 비국소 tail을 package validation 항목으로 만들었다.
  Owner support의 \(M\)-orthogonal projector와 \(\rho_{r,\mathrm{tail}}\)을 정의하고,
  큰 tail에서만 support 확대, overlap group 병합 또는 locality-constrained basis를 검토한다.
- Pure cached-compliance의 blind spot을 확인하기 위해 strain, curvature와 runtime-observable truncation을
  development-split scale로 무차원화한 deterministic hybrid를 조건부 ablation으로 추가했다.
  Feature 생성 비용도 selector latency에 포함한다.
- Corrector 비용을 작은 reduced solve 하나로 축약하지 않고 corrected geometry/Jacobian,
  active/halo aerodynamic reevaluation, delta construction과 reduced solve로 나눠 측정한다.
- Core quantitative dynamics suite는 short cantilever strip, rectangular long flag,
  triangular pennant와 curtain strip으로 고정했다. Hanging cloth는 secondary topology/attachment stress test,
  paper-like sheet와 single leaf/petal은 transfer 또는 qualitative 결과로 내렸다.
  MC1의 5--10개 independent-GS representation probe는 별도 범위로 유지한다.

### 부분 수용·반박·보류한 내용

- Compliance, strain, curvature와 truncation을 단위가 다른 raw score 그대로 더하는 식은 거부했다.
  정규화된 hybrid만 ablation하고 pure compliance와 차이가 없으면 V0 mainline을 유지한다.
- Coordinate/rank budget 비교는 진단으로 수용하지만 Gate B/C의 primary fairness 기준으로는 거부했다.
  Local support, cross block, assembly, factor/cache 비용이 달라 동일 coordinate 수가 동일 compute가 아니므로,
  same measured p95에서 velocity RMSE를 primary로 하고 trajectory와 PSD/spectrum 및 전체 Pareto를 함께 본다.
- 리뷰 도식의 `Local solve 후 assembly` 순서는 거부했다. Canonical 실행 의존성은
  patch proposal, unique-anchor assembly, joint conservation/Global complement, generalized projection,
  single coupled Local solve, delta Global corrector 순서다.
- 결과 전에 working title을 즉시 `Selective`로 낮추라는 처방은 보류했다.
  Deterministic selector도 held-out Oracle correlation/regret/activation-delay와 quality--latency를 통과하면
  `Error-Triggered`를 방어할 수 있으며, 실패할 때만 fallback title로 축소한다.
- `directly renderable`은 renderer 연결 I/O 계약이지 `interactive` 또는 `real-time` 주장이 아니다.
  Renderer 포함 end-to-end p95 전에는 interactive 표현을 쓰지 않고, CGF/TVCG venue 판단도 증거 뒤로 미룬다.

### 변경 범위와 검증

- 이번 요청 범위에 따라 canonical 아이디어 스케치 TeX/PDF/bundle과 이 session note만 갱신했다.
  구현 체크리스트는 수정하지 않았으며, fixed-cardinality 명칭과 새 계측 항목의 후속 동기화가 필요하다.
- `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error` 빌드가 성공해 86쪽 PDF를 생성했다.
- 최종 log에는 LaTeX error, undefined control sequence/reference/citation, multiply-defined label과
  Overfull box가 없다. 남은 경고는 CJK italic font substitution과 Underfull box뿐이다.
- Tail, runtime dependency, normalized hybrid, fixed-cardinality, corrected-aero, core asset,
  coordinate-budget 반박, title과 interactive/venue 보류 박스를 PDF에서 rasterize해 clipping 없이 확인했다.
- 스케치 bundle은 최신 TeX/Bib/PDF 세 파일만 포함하도록 다시 생성했다. `unzip -t`를 통과했고
  archive 내부 세 파일의 SHA-256이 working file과 각각 일치한다.

## 2026-08-14 후속 revision review v3 수용·반박 반영

후속 보고서 `wind3dgs_revision_review_report_2026-08-14_v3.pdf`를 당시 86쪽 canonical
아이디어 스케치와 대조했다. V3는 최신 revision의 fixed-cardinality selector, complement tail,
unique-anchor Schur, corrected-minus-predictor feedback과 H/F/G 독립 go/no-go를 정확히 읽었다.
새 기능을 더 늘리기보다 구현 전에 비어 있던 structural operator와 backend source of truth를 닫는 데 집중했다.

### 수용한 내용

- Relation-local metric MLS, normalized area partition, typed mass/material/attachment를 이용해
  (M,K,C) assembly를 구현 가능한 수준으로 명시하고, backend/integrator를 Track A0 전에 동결했다.
- Rest-linear (K_{\mathrm{lin}},C_{\mathrm{lin}})은 small-strain calibration, mode와 baseline에,
  objective vector-projective (K_{\mathrm{proj}},C_{\mathrm{proj}})은 finite-rotation runtime에 쓰도록 분리했다.
- Canonical V0는 one-shot descriptor-weighted local rotation과 implicit midpoint로 고정했다.
  같은 projective energy를 local--global iteration으로 수렴시킨 결과를 accuracy reference로 추가했다.
- Global complement tail은 보편 threshold가 아니라 asset/load별 downstream trajectory·PSD degradation과
  calibration하고, independent-GS package consistency, p99/cache/factor telemetry를 유지했다.
- 첫 정량 material/geometry 범위를 flat/shallow-rest strip, flag, pennant, curtain의
  asset-uniform isotropic \(\nu=0\) family로 한정했다. Nonzero Poisson ratio, anisotropy와 pre-curved leaf/shell은
  qualitative/후속 범위로 내렸다.

### 부분 수용·반박·감수한 내용

- V3의 raw (B^\top D B)+positive coefficient만으로는 finite-rotation objectivity가 보장되지 않는다는
  문제는 수용했지만, 제시된 generic corotational skeleton을 그대로 쓰지는 않았다.
  Scalar rest-frame strain target은 strained state에 rigid rotation을 중첩할 때 channel을 섞으므로,
  world-vector descriptor와 block-isotropic weight를 쓰는 objective projective backend로 교체했다.
- Backward Euler를 canonical integrator로 쓰라는 처방은 MC2의 high-frequency flutter/PSD를
  구조적으로 감쇠하므로 거부했다. BE는 robustness/debug reference로만 남기고 implicit midpoint를 유지한다.
- Review diagram의 Local solve 후 conservation/complement assembly 순서는 거부했다.
  Patch proposal, unique-anchor joint assembly/complement, generalized projection,
  single coupled Local solve, delta Global corrector 순서를 유지한다.
- 모든 baseline을 점별 strict dominance해야 한다는 해석은 거부했다. Same-p95 error,
  matched-quality speedup과 multi-object throughput을 포함한 Pareto 비지배 영역으로 판정한다.
- `interactive system`과 선제적 CGF/TVCG venue 판정은 end-to-end p95와 결과가 나오기 전까지 보류한다.

### 추가로 닫은 구현 계약

- Local rotation은 runtime energy와 같은 descriptor/weight의 exact stationary local step으로 정의했다.
  Relative-rate spin은 relation rotation history의 matrix-log 대신 predictor descriptor의
  \(\Omega^\eta\)-weighted skew least squares와 Moore--Penrose 해로 계산한다.
- One-shot frozen target에서는 predictor에서만 internal torque stationarity가 exact하므로,
  최종 torque residual은 splitting error로 측정하고 tolerance 실패 시 iteration/backend 재설계로 보낸다.
- Hard support는 V0에서 coordinate-selection constraint로 제한하고 force-covector projection을 명확히 했으며,
  moving compliant target mapping과 attachment target/lift ownership을 한 번만 적용하도록 닫았다.
- Surface/aero first pass는 causal half-step geometry와 frozen endpoint velocity를 사용한다고 명시했다.
  Homogeneous structural update는 implicit midpoint지만 one-pass exogenous load/coupling 전체를
  자동으로 2차 정확하다고 주장하지 않고 true-midpoint/iterated reference와 비교한다.
- Global structural feedback은 predicted-midpoint coupling을 먼저 소비한 뒤 Local update의
  actual-midpoint coupling과의 delta만 factor 2와 함께 corrector RHS에 넣도록 수정했다.
- Oracle topology/relation privilege와 native teacher operator ceiling을 분리하고,
  E1 eigenspectrum은 \((M_f,K_{\mathrm{lin},f})\), canonical runtime은 force action/objectivity/rollout으로 평가한다.

### 변경 범위와 검증

- 사용자 요청에 따라 canonical 아이디어 스케치 TeX/PDF/bundle과 이 session note만 갱신했다.
  구현 체크리스트는 수정하지 않았다.
- `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error` 빌드가 성공해 98쪽 PDF를 생성했다.
- 최종 log에는 LaTeX error, undefined control sequence/reference/citation, multiply-defined label과
  Overfull box가 없다. 남은 경고는 CJK italic font substitution, microtype와 Underfull box뿐이다.
- 수학/force-ledger와 전역 source-of-truth 재감사에서 남은 구현 차단 이슈가 없음을 확인했다.
- 스케치 bundle은 최신 TeX/Bib/PDF 세 파일만 포함하도록 다시 생성했으며,
  `unzip -t`와 archive-to-working-file SHA-256 일치를 확인했다.

## 2026-08-14 후속 revision review v4 수용·반박 반영

후속 보고서 `wind3dgs_revision_review_report_2026-08-14_v4.pdf`를 당시 98쪽 canonical
아이디어 스케치와 대조했다. V4는 normal-curvature bending pollution, support-limited
Global complement feasibility, absolute relation confidence와 rest-linear/projective closure를
핵심 위험으로 지적했다. 사용자 요청에 따라 유효한 진단은 본문 계약과 P0 kill test에 편입하고,
과도하거나 수학적으로 불완전한 처방은 해당 절의 `V4 리뷰 판정` 코멘트에서 반박·제한했다.

### 수용한 내용

- In-plane second derivative까지 bending으로 벌점화하던 문제를 수용해, predictor normal projector와
  normal-curvature target을 쓰는 frozen-target operator로 교체했다. One-shot final state의 tangential
  residual은 0이라고 주장하지 않고 splitting error로 측정하며 two-step/converged reference와 비교한다.
- Normalized area share와 별도로 relation-validity probability 및 count/conditioning-aware absolute
  anchor confidence를 두었다. Anchor-level teacher target, NLL/Brier calibration, ECE/AUPRC,
  coverage/false-accept hard gate와 support 확대/fallback/rejection 경로를 명시했다.
- Rest-linear operator와 runtime projective action의 small-strain closure를 raw Hessian이 아니라
  locally refit한 diagnostic energy와 실제 held-target action/tangent 기준으로 검사한다.
  Energy/force, skew, PSD, modal subspace와 common-probe error를 P0 hard gate로 묶었다.
- V0 aerodynamic lifting은 fixed (T_s^{A\rightarrow S})와 fixed \(\Phi_s\)로 확정했다.
  Current/corotational Jacobian은 optional nonlinear branch로 내리고 reconstruction 비용을 별도 보고한다.
- High-curvature/low-load, topology/operator negative control, cross-group excitation와 fast traveling/reversing
  gust를 selector blind-spot fixture로 추가했다.
- Support-restricted Global routing, current-geometry conservation과 hard-support companion reaction을
  channel별로 정의하고, post-corrector total reaction과 free-space ROM residual을 Module 12에서 함께 복원한다.

### 부분 수용·반박·감수한 내용

- Normalized relation-weight raw sum을 absolute confidence로 쓰라는 처방은 거부했다. Raw scale은
  common logit shift와 candidate count에 비식별적이므로 별도 calibrated validity head를 사용하고,
  confidence를 stiffness에 곱해 불확실성을 재료 연화로 숨기지 않는다.
- Sparse support에서 \(\operatorname{rank}(\Phi_t)<r_g\)이면 항상 실패라는 처방은 거부했다.
  실제 RHS consistency, nonzero-spectrum conditioning과 force amplification을 검사하고, 실패할 때만
  halo를 확대한다. Ridge로 infeasibility를 숨기지 않는다.
- External remainder는 support-restricted routing 뒤 이미 feasible witness이므로 같은 equality를
  redundant Schur로 다시 풀지 않는다. Structural/reaction 또는 추가 low-row constraint에만
  augmented small-dual Schur를 사용한다.
- Hard reaction이 reduced trajectory의 full free-DOF residual까지 없앤다는 해석은 거부했다.
  정확한 분해에 (r_{\mathrm{ROM}})을 남기고 Global/Local virtual-work residual과 teacher reaction error를 보고한다.
- Corrected-aero delta는 conservative transpose scatter로 anchor field에 저장하고 post-step applied-force
  ledger에 정확히 한 번 포함한다. Structural delta는 final held action에 이미 들어가므로 별도 중복하지 않는다.

### 조건부 missing-force와 실행 계약 보강

- Hard structural teacher label과 runtime assembly가 동일한 metric, zero-reaction reference와 augmented
  constraint를 사용하도록 맞췄다. Raw-head target과 post-assembly \((y^*,\rho_h^*)\) target을 분리해
  double projection을 막았다.
- Channel별 generalized-force split consistency를 추가하고, force/reaction type이 다른 aero와 structural
  defect를 source가 불명확한 single total head로 합치는 경로는 no-go로 정했다.
- Generalized-force-only lifting은 channel generalized target뿐 아니라 별도 conservation target이 있을 때만
  후속 optional 경로로 허용한다. Hard structural generalized fallback은 first-paper에서 no-go다.
- Force metric (W_f\), support restriction (W_{f,t}), graph smoother (L_f)의 차원·boundary·version을
  명시하고 smoothing/SSIM weight 기호를 분리했다. Raw channel leakage와 post-assembly solver diagnostic도 분리했다.
- Canonical dependency는 proposal, unique-anchor assembly, support route/conditional solve,
  channel generalized projection, single coupled Local solve, delta Global corrector 순서로 유지한다.

### 변경 범위와 검증

- 이번 요청 범위에 따라 canonical 아이디어 스케치 TeX/PDF/bundle과 이 session note만 갱신했다.
  구현 체크리스트는 수정하지 않았으며 V4 backend와 아직 미동기화되어 있다. 특히 checklist의
  `Current Global Jacobian`은 canonical fixed \(\Phi_s\) 계약과 충돌하므로 후속 동기화가 필요하다.
- `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error` 빌드가 성공해 111쪽 PDF를 생성했다.
- 최종 log에는 LaTeX/package error, undefined control sequence/reference/citation,
  multiply-defined label, rerun warning과 Overfull box가 없다. 남은 경고는 microtype/CJK font substitution과
  Underfull box뿐이다.
- V4 operator, absolute-confidence target, force label/lifting, runtime dependency, reaction/ROM residual,
  conditional-F loss와 Step 0 페이지를 rasterize해 clipping/overlap이 없음을 확인했다.
- 스케치 bundle은 TeX/Bib/PDF 세 파일만 포함하며 `unzip -t`를 통과했다. Working file과 bundle 내부
  SHA-256은 TeX `7d48655e...73f1`, Bib `c2e3df67...660a`, PDF `327bee6d...4e44`로 각각 일치한다.

## 2026-08-15 후속 revision review v5 수용·반박 반영

후속 보고서 `wind3dgs_revision_review_report_2026-08-15_v5.pdf`를 당시 111쪽 canonical
아이디어 스케치와 구현 체크리스트에 대조했다. V5는 구조·routing·reaction 설계를 뒤집는 리뷰가 아니라
architecture freeze와 Step 0 착수를 승인하면서 문서 실행 계약을 정리하라는 성격이었다. 새 수학적
fatal flaw는 없었고, 유효한 운영 보강은 canonical sketch와 stale checklist 양쪽에 반영했다.

### 수용한 내용

- Closure 실패를 action-validity와 rest-linear-agreement로 분류하고 원인별 fallback을 명시했다.
  Canonical V0는 두 판정을 모두 통과해야 하며, one-shot splitting만 실패할 때 two-step/converged
  target fit을 검토하고 action 자체가 실패하면 backend를 재설계한다.
- Relation confidence는 object-disjoint split과 density/valence/conditioning 층화 calibration으로
  검증하며, operator hard check를 대체하지 않는 triage/fallback signal로 유지했다.
- Support routing의 consistency, conditioning, force/reaction amplification과 halo expansion에 대해
  development equivalence-class/bin cap을 동결하고 test-time cap 초과는 reject 또는 선언된
  always-on fallback으로 보낸다.
- 111쪽 derivation을 canonical equation source로 유지하면서, stable `eq:` label, typed API/owner,
  schema version과 failure policy만 담는 파생 V0 contract를 TD01 artifact로 지정했다.
  `v0_contract_manifest.json`과 stale label/API/owner/hash negative test를 TD01 완료 조건에 추가했다.
- 구현 체크리스트를 metric-MLS/projective operator, implicit midpoint, fixed \(T_s,\Phi_s\),
  support-restricted routing, channel별 force labels, hard/free/compliant reaction, actual-midpoint corrector,
  cached-compliance/decay와 P0 K00--K24 계약에 동기화했다.

### 부분 수용·반박·감수한 내용

- 모든 closure 실패에 action-tangent basis, iteration, state-dependent tangent를 고정 순서로 적용하라는
  해석은 거부했다. 특히 modal/probe disagreement가 난 뒤 basis만 바꿔 같은 hard gate를 통과시키는 것은
  논리적으로 불가능하다. Action-basis는 기존 MC1 rest-linear eigenspace claim을 철회하고
  \(\Phi,\Psi\), cross block과 cache를 모두 다시 만드는 별도 versioned pivot으로만 허용한다.
- 20--30쪽 독립 normative 문서를 새 source of truth로 만드는 처방은 drift 위험 때문에 채택하지 않았다.
  파생 contract는 canonical 수식을 복제하지 않고 label/API assertion만 보유한다.
- Formal Track B0와 learned calibration은 Step-0 gate 뒤에 시작하지만, coordinate/relation schema,
  valence/rank, area/mass와 close-layer false edge를 보는 최소 independent-GS probe는 Step 0과 병렬 허용한다.
- Step 0에서 아직 학습되지 않은 K21 calibration 완료를 요구하지 않는다. Step-0 gate는 K00--K09의
  deterministic 해당 항목, K20, K22--K23과 K21 schema/synthetic fallback fixture이며,
  learned K21은 Track B0/E1, selector K24는 downstream stage에서 판정한다.
- Structural backend는 MC1을 가능하게 하는 핵심 구현이지만 독립 headline novelty로 격상하지 않고,
  venue/interactive 표현도 결과와 end-to-end p95가 나오기 전까지 보류한다.

### 변경 범위와 검증

- Canonical 아이디어 스케치와 구현 체크리스트의 TeX/PDF/bundle, 이 session note를 갱신했다.
  다른 dirty worktree 변경은 건드리지 않았다.
- 두 문서 모두 `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 빌드했다.
  스케치는 114쪽, 체크리스트는 46쪽이다.
- 최종 log에는 LaTeX error, undefined control sequence/reference/citation, multiply-defined label,
  rerun warning과 Overfull box가 없다. 남은 것은 Underfull과 font/hyperref 계열 비치명 경고뿐이다.
- Closure, canonical contract, critical path와 TD01 manifest 페이지를 rasterize해 clipping/overlap이
  없음을 확인했고, `git diff --check`와 PDF의 `??` 검사도 통과했다.
- 스케치 bundle은 TeX/Bib/PDF 3개, 체크리스트 bundle은 TeX/PDF 2개만 포함한다.
  두 archive 모두 `unzip -t`를 통과했고 working/archive SHA-256이 일치했다.
  최종 working hash는 sketch TeX `9b5e9ab5...c223a`, Bib `c2e3df67...4660a`,
  sketch PDF `8d3ab9bf...af910`, checklist TeX `8a64009e...90551`,
  checklist PDF `ac9180c3...29523`이다.

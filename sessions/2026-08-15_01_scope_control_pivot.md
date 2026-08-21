# 2026-08-15 01 scope-control Minimal V0 전환

## Context

여러 차례의 외부 리뷰를 반영하는 과정에서 응용 범위는 제한했지만,
physics-only V0 내부에 objective normal-curvature backend, support-restricted KKT/Schur,
reaction ledger, same-frame corrector와 confidence calibration까지 들어가 구현·검증 범위가 커졌다.
CG 논문의 실제 목표인 visual coherence, stability와 measured quality--latency보다
engineering-grade correctness package가 앞서기 시작했다는 scope-control 판정을 내렸다.

참고한 `wind3dgs_scope_control_review_report_2026-08-15.pdf`의 핵심 진단은 수용했다.
다만 objectivity, rest recovery, basis rank, state continuity와 covariance SPD까지 버리자는 뜻으로
확대하지 않았으며, 이 항목들은 visible failure를 막는 최소 안전 계약으로 유지했다.

## Decisions

### Minimal V0 core

1. Independent static GS에서 sparse oriented anchor scaffold, area/mass와 fixed binding을 만든다.
2. User material/hard attachment와 검증된 simple corotational/PD backend 하나를 사용한다.
3. 모든 anchor에서 current-surface quasi-steady aero를 frame당 한 번 계산한다.
4. Deterministic fixed-cardinality selector, Active/Decay와 persistent Local slots를 사용한다.
5. 하나의 external anchor force를 `[Phi, Psi_S]`에 직접 투영하고 cross block을 포함한
   single coupled implicit-midpoint step을 수행한다.
6. Anchor deformation을 Gaussian mean/covariance로 affine transport한다.

### Core에서 제거·강등

- H/F/G, Teacher-FSI
- generalized-to-nodal lifting, patch-force KKT/Schur와 exact support reaction routing
- same-frame aerodynamic/structural corrector
- calibrated confidence network와 full statistical calibration suite
- custom normal-curvature/action-closure backend와 exact modal/energy/wrench hard gates
- profiler가 필요성을 보이기 전의 incremental factorization, block-PCG와 dynamic cache

위 모듈은 core benchmark에서 visible artifact 제거, claim-invalidating failure 해결 또는
same-p95/matched-quality Pareto 개선을 보인 경우에만 하나씩 다시 연다.

### 유지한 최소 correctness

- unit/metric convention, positive area/mass와 close-layer rejection
- rigid-transform sanity, zero-wind rest recovery와 bounded rollout
- Global--Local mass complement, combined-basis rank/condition과 cross blocks
- switch 시 energetic Local state reset 금지와 bounded Active/Decay policy
- affine/rigid transport, covariance SPD와 clamp/fallback 기록
- same-anchor baseline, synchronized p50/p95/p99와 actual active/solve rank

### Research gates

- Gate A: independent-GS scaffold/response feasibility
- Gate B: strong Global-rank 대비 Oracle complementary Local necessity
- Gate C: deterministic selector를 포함한 actual deployable Pareto gain

MC1과 MC2는 독립적으로 축소할 수 있고, SYS는 supporting system/scaling evidence로 둔다.
Working title은 evidence 전 `Selective`로 낮췄으며 `Error-Triggered`를 주장하지 않는다.

## Archived predecessor

Scope pivot 직전의 current working-tree full design을
`backup/full_design_before_minimal_v0_2026-08-15/`에 원래 이름으로 동결했다.
114쪽 sketch, 46쪽 checklist, bibliography와 두 delivery bundle을 보존했으며
archive README에서 non-canonical/완료상태 비승계를 명시했다.

보존 시 SHA-256:

- full sketch TeX: `9b5e9ab5e736a1bbff37f533b690a76b8964085cece6db22bfa13e73f85c223a`
- bibliography: `c2e3df67f86813a95b4fb4570e913130b93825cf3c43923526c83fc60674660a`
- full sketch PDF: `8d3ab9bf9c3f5bdf8babc220821515f4d33967b53885e2b6c43ea80b22faf910`
- full sketch bundle: `e09d1ed7ed70662a365a00587d8fa9a8e66f03e0c3c4bed530f43e878a1b3e25`
- checklist TeX: `8a64009e355daae9dd98860e06cc0ef06ac233df826cb578475123f049e90551`
- checklist PDF: `ac9180c331459353c49d12fb528aabca302e24ca9b436d17515ee26127329523`
- checklist bundle: `5b720d604cb8661294ce34e7939b092afa5c650133860168e46b0988230b7584`

## Changed Files

새 canonical set:

- `3dgs_topology_distilled_selective_global_local_wind_dynamics_2026-08-15.tex`
- `refs_topology_distilled_selective_global_local_wind_dynamics.bib`
- `implementation_checklist_topology_distilled_selective_global_local_wind_dynamics_2026-08-15.tex`
- 각 canonical PDF와 delivery bundle

Index/policy:

- `README.md`: 새 title/files, Minimal V0 scope와 predecessor archive pointer
- `.gitignore`: current Minimal V0 PDF/bundle과 frozen predecessor allowlist
- `backup/full_design_before_minimal_v0_2026-08-15/README.md`

## Verification

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 빌드했다.
- Sketch는 17쪽, checklist는 7쪽이며 두 PDF 모두 non-empty A4다.
- 두 log에서 LaTeX error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 overfull box가 없음을 확인했다.
- Sketch의 title/scope, local basis, coupled solve, gate/deferred table, bibliography 페이지와
  checklist의 title, hard-gate, deferred/DoD 페이지를 raster inspection했다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개이며
  `unzip -t`를 통과했다. Bundle 내부 entry와 working file의 SHA-256이 모두 일치했다.
- Frozen predecessor 두 bundle도 `unzip -t`를 통과했고 위 보존 hash와 일치한다.

최종 current SHA-256:

- sketch TeX: `754de8507cf9489b126e91fee9f2409235aa27e9114e758287eb15538eac0f04`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `c0e904f349915f9d8ddf4a7558e369136d7c82f01df59f5d7f7d45d549aa6bbb`
- sketch bundle: `27ff25bcc470435d131a7ce60c4a2c2c9a3706326e53bc2e992aa029bc1bf460`
- checklist TeX: `b65dc033c2c952fe0b1f2f2d078ebb6f275bae6f4897c40230a86c7978b07206`
- checklist PDF: `8310a80958b982b1e48655a5ac6ff403ffde4f3a53630db3b42b37fa8f1b7a19`
- checklist bundle: `9684f198f6344b9185ba22251408fe44a955947a13e5969ea72b114df79fb05f`

## Next

- `code/`와 `experiments/`의 기존 완료 상태는 새 S0--S7을 통과하기 전까지 승계하지 않는다.
- 후속 code-side contract와 experiment matrix는 새 sketch의 stable label과 Gate A/B/C에 맞춘다.
- 첫 실행은 Oracle strip V0-00/V0-01이며, 대규모 topology training/H/F/G 작업은 시작하지 않는다.

## 2026-08-17 canonical Minimal V0 리뷰 후속 반영

올바른 current 17쪽 canonical을 검토한
wind3dgs_minimal_v0_canonical_review_report_2026-08-16.pdf를 기준으로 다시 감사했다.
리뷰의 구현 모호성은 수용했지만, solver family·calibration·dataset을 늘리는 방식은 택하지 않았다.
기존 S0--S7 여덟 gate와 deferred registry를 유지하면서 다음 계약만 구체화했다.

- Structural backend는 edge-opposite-triplet-v1 relation schema와
  one-local-step fixed-Hessian PD-style projector 하나로 고정했다.
  Stretch edge, opposite-triplet bending, owner-area weight, boundary skip/coverage와
  deterministic replay artifact를 명시했다.
- SI는 직접 지정한 areal density, nondimensional branch는
  \((L_0,M_0,T_0,\widehat\rho_A)\)와 별도 preset을 사용한다.
  두 branch를 한 package/run에서 섞지 않고 동일 solver code path로 정규화한다.
- Fixed \(\Delta t\), internal substep 1, wind/gust/force/clamp bound를
  operating envelope와 manifest에 저장하고 canonical run의 OOD는 fail-fast한다.
- Selector의 desired set과 실제 admitted Active set을 분리했다.
  Decay 재활성화의 atomic swap, dwell/fade/hold, Decay capacity,
  dimensionless energy/displacement/velocity/pop proxy와 release rule을 동결했다.
- Current-surface normal/area는 valid rank-2 cross product를 exact normalize하고,
  numerical floor는 분모 epsilon이 아니라 explicit invalid-frame gate로 사용한다.
- Gaussian transport는 positive normalized MLS weight와 weighted centering을 사용하고,
  \(J^0/J^t\) relative tangent map과 fitted normal extension으로 identity/proper-rigid motion을
  정확히 재현한다. Covariance에는 unconditional \(\varepsilon I\)를 더하지 않고,
  declared eigen bound 밖에서만 conditional clamp한다.
- Oracle Gate B는 V0-03 직후, deployable Gate C는 V0-04 직후 실행한다.
  직접 wind-responsive 3DGS baseline은 Gate A/B/C가 살아남은 paper-evidence 단계에서
  비교 가능한 공개 구현 한 개만 선택한다.

Held-out geometry, real capture, view-count sweep와 여러 direct 3DGS adaptation은
Minimal V0/P0로 승격하지 않았다. Learned gate, KKT/Schur, support reaction,
same-frame corrector와 confidence calibration도 deferred 상태를 유지한다.

### 후속 검증

- 두 canonical TeX를
  latexmk -g -xelatex -interaction=nonstopmode -halt-on-error로 재빌드했다.
- Sketch는 21쪽, checklist는 9쪽의 non-empty A4 PDF다.
- 두 log에서 LaTeX error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 Overfull box가 없음을 확인했다.
- Setup/backend, surface/aero, Active/Decay, relative affine transport,
  safety table와 checklist S0--S6 페이지를 raster inspection했다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다.
  두 bundle 모두 unzip -t를 통과했고 내부 entry hash가 working file과 일치한다.
- 마지막 수학 감사에서 rigid objectivity/rest-zero, SI weight 차원,
  state transition, exact surface normalization, relative transport와 covariance identity에
  치명적 또는 major blocker가 없음을 확인했다.

2026-08-17 current SHA-256:

- sketch TeX: 6661c4d8a5f120bf0a75e86dc01aab9a28ae65219b8ffc8f2174d4ccf8ba894b
- bibliography: 067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d
- sketch PDF: 57fc302aeda011a4db815f8a1e1dccdff4706031a67d5238d7bed972d0eae7ad
- sketch bundle: c65fda86ac8a66db55416d24adc1c4588e42546a92d255b32df20b4aa5bc4c99
- checklist TeX: 5050f95eadd3a18aeb8cd00fe210934d2481c3d5bbcb633e58ef084705669e1b
- checklist PDF: 43c7360d899393a11a3328e2f61a72bf473e5c74e441cec219cffe88bdf1368e
- checklist bundle: d86d0aa2f251f8afe9ae299cd3fc3c7e15e95deb14c05639811779d17d9a0a49

이 반영은 아이디어 문서 계약 갱신이며 code/experiment 완료 상태를 새로 만들거나 승계하지 않는다.

## 2026-08-17 canonical 리뷰의 남은 P0 계약 동결

`wind3dgs_minimal_v0_canonical_review_report_2026-08-17.pdf`를 current Minimal V0
sketch/checklist와 대조했다. 리뷰가 지적한 세 구현 공백만 닫고, 새 solver·network·gate는
추가하지 않았다.

- Global basis는 free-coordinate rest generalized eigenmode 하나로 고정하고
  `basis_type=generalized_eigen_v1`을 package/manifest/hash에 저장한다.
  POD는 Gate A/B/C 이후의 non-blocking ablation으로만 허용한다.
- 정량 core는 `core_scope=flat_rest_v0`의 cantilever strip과 rectangular flag로 제한한다.
  Shallow/pre-curved asset은 Gate 이후 qualitative/secondary backlog로 내렸다.
  다만 irregular flat sampling에서는 opposite-triplet rest descriptor가 정확히 0이 아닐 수 있으므로
  planarity/tangential-residual cap과 reject rule은 유지한다. 이를 해결하려고 새 shell backend를
  추가하지 않는다.
- Current surface와 Gaussian binding은 positive normalized weights, weighted centering,
  fixed rest coordinate를 공유하는 `tangent_fit_version=tangent_fit_v1`으로 고정했다.
  Aerodynamic step (t)는 (s=t-1) fit만 소비하여 one-frame staggered coupling을 유지한다.
- Surface/aero rank·condition·normal 퇴화는 canonical quantitative run에서 fail-fast한다.
  리뷰가 제안한 closest-rigid fallback은 존재하지 않은 aerodynamic load를 만들 수 있어 수용하지
  않았고, renderer-only visual fallback과 분리했다.
- 세 signature는 기존 package/run hash와 S0/Definition of Done에 포함했다. 별도 artifact 체계나
  아홉 번째 hard gate는 만들지 않았다.
- V0-03 직후 preliminary dynamics-only Gate C, V0-04 직후 transport/render를 포함한 final
  deployable Gate C를 실행한다. V0-00의 method/contract freeze와 V0-05의
  claim/research-route freeze를 구분했다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 재빌드했다.
- Sketch는 22쪽, checklist는 9쪽의 non-empty A4 PDF다.
- 두 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 Overfull box가 없음을 확인했다.
  Sketch bibliography의 Underfull 1건과 checklist의 CJK italic font substitution만 남았으며
  둘 다 내용·레이아웃을 깨뜨리지 않는 비차단 경고다.
- Signature/Global basis/current-surface/구현 순서와 checklist S0/S3/S4/DoD 페이지를
  raster inspection했고 clipping·겹침을 발견하지 못했다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다.
  두 bundle 모두 `unzip -t`를 통과했고 내부 entry hash가 working file과 일치한다.
- 최종 읽기 전용 감사에서 세 P0, manifest/DoD, surface fail-fast, 두 Gate-C 시점 사이에
  blocker나 stale 선택지가 없음을 확인했다.

2026-08-17 최종 SHA-256:

- sketch TeX: `de7698f95b326ec2c234b3f401063a83def125dfc1deb47667d9a106d94acc7b`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `a2d63346f18980592e5b68a51eaa2b341119ad672ca92595782fcfd414d9c6e5`
- sketch bundle: `f06b83d5f227d2a5b5cc1048feebd7a505adcdaca4004940895b10b236c30479`
- checklist TeX: `df490844fc3e713eb20ca919ec6a51794a0a5e6e89bd9202f7c70117935d2d30`
- checklist PDF: `0798c3c003f332bbcdcc14a23e69682daa36162433fac3f6debd1af28543a675`
- checklist bundle: `5ba0c18553bfdd2442ecf08e529881b0c4242dad233fa5e4eb3bff356cc5dc8b`

이번에도 code/experiment 완료 상태는 생성하거나 승계하지 않았다.

## 2026-08-18 P0 verification 후속 리뷰 반영

`wind3dgs_minimal_v0_p0_verification_report_2026-08-17.pdf`를 current Minimal V0
canonical sketch/checklist와 대조했다. 보고서의 P0-1은 과거 39쪽 checklist를 대상으로 한
stale 지적이며, 현재 Minimal V0 checklist는 이미 H/F/G, KKT/Schur와 same-frame corrector를
core에서 금지하고 있으므로 구 계약을 다시 들여오지 않았다. 실제 남은 두 P0 공백과 작은
재현성 계약만 다음처럼 닫았다.

- Gate 실행 전에 canonical package/sequence attempt 집합을 사전 등록한다.
  Operating envelope 밖으로 사전 판정한 OOD만 분모에서 분리하고, in-envelope package reject,
  rank collapse, divergence, non-finite state, unresolved transport/covariance failure는 method failure로
  고정 분모에 남긴다. $R_{\mathrm{accept}}$, $R_{\mathrm{success}}$, first-failure reason과
  conditional quality를 함께 보고하며 Gate A/C failure cap을 development split에서 동결한다.
- Renderer-only affine fit 실패는 centered weighted proper Kabsch/Procrustes로 처리한다.
  Deterministic SVD/tie-break, $\det R=+1$, relative numerical rank
  $\operatorname{rank}_{\tau_K}$, singular-value ratio/planarity와 normalized residual을 모두
  통과한 경우에만 fallback-marked renderer output을 허용한다. $\tau_K$와 모든 threshold,
  fallback-rate cap은 binding/run manifest와 hash에 저장한다. Surface/aero consumer는 이 fallback을
  사용하지 않고 기존 fail-fast 계약을 유지한다.
- Irregular flat rest descriptor의 $\delta_{ia}^{\mathrm{rest}}$는 report-only diagnostic으로만
  추가했다. 새 hard gate, reject trigger 또는 shell solver를 만들지 않았다.
- SI branch의 position/covariance/area scale과 Rayleigh $\alpha,\beta$ 단위 및 nd mapping을
  명시했다.
- Canonical quantitative appearance는 `appearance_mode=degree0_v0`로 고정했다. Opacity와 DC color는
  유지하고 higher-order SH canonical evaluation은 끈다. Analytic local-frame SH는 Gate 이후의
  optional ablation으로만 남겼다.

S0--S7 8개, milestone 7개와 deferred registry 8개를 유지했으며, 새 solver, network, stage 또는
hard gate를 추가하지 않았다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 재빌드했다.
- Sketch는 24쪽, checklist는 11쪽의 non-empty A4 PDF다.
- 두 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 Overfull box가 없음을 확인했다.
  Sketch bibliography의 기존 Underfull 1건만 남았으며 내용·레이아웃 비차단이다.
- 평가 분모, renderer proper-Kabsch/rank threshold, S6/S7, 구현 순서와 DoD 페이지를 raster
  inspection했고 clipping, 겹침, 깨진 수식과 `??`를 발견하지 못했다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다.
  두 bundle 모두 `unzip -t`를 통과했고 내부 entry hash가 working file과 일치한다.
- 최종 수학·계약 감사에서 Kabsch 부호/차원/reflection/degeneracy, OOD/failure 분모,
  SI/Rayleigh, appearance와 scope 사이에 남은 blocker가 없음을 확인했다.

2026-08-18 최종 SHA-256:

- sketch TeX: `b53fb7903f28d7db2eb735e3a5f93632bd59b056359cbb6c2b29f91a06d19b1e`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `d42eaa38029ac111eb79ff19b77c8dcc42cd604d3dd423cd8221c8b1c9719c54`
- sketch bundle: `ef4d1873d23783fbda62620659a0331de96f68f0bd0112f1be9a38efb9da6658`
- checklist TeX: `8703065380e03457fb2262357b9895dbc087bcf4cc9581e3b2de69559569172c`
- checklist PDF: `8453495419a6ac65fe4d5457695a43f3de13cf118074ae30f948cf6f3dcecb75`
- checklist bundle: `2196aea825644f19a8823e764eca11d61b7ba1bca4dbd06d1144b908a8145630`

이번 반영도 아이디어/실행 계약 갱신일 뿐 code/experiment 완료 상태를 생성하거나 승계하지 않았다.

## 2026-08-18 Minimal V0 revision recommendation 후속 반영

`wind3dgs_minimal_v0_revision_recommendation_report_2026-08-18.pdf`를 current Minimal V0
canonical sketch/checklist와 대조하고, 수용할 진단은 최소 계약으로 반영하되 과도하거나 stale한
처방은 명시적으로 제한했다.

- Irregular-flat relation에서 nonzero rest descriptor를 sphere orbit에 투영하면 normal quadratic
  stiffness가 사라질 수 있다는 진단을 수용했다. Bending projector를
  `bend_projection_version=corotated_tangent_plane_v2` 하나로 교체하고, rest/predictor 양쪽에서
  동일한 versioned tangent fit과 orthonormalization을 사용한다. Predictor-frozen target과 fixed
  Hessian/one-step PD 계약을 유지해 새 runtime stage나 local/global iteration은 추가하지 않았다.
  Unit projector 검사는 fitted frame을 고정한 quadratic-response test로, integrated fixture는
  rest/proper-rigid exactness와 positive non-collapsed normal response로 분리했다.
- In-plane tangent gauge 공백을 수용했다. Core hard attachment에서 deterministic
  sign-invariant unoriented line을 만들고 eigengap/projection degeneracy는 package reject로 처리한다.
  `tangent_gauge_version=attachment_line_v1`을 package hash에 포함하며, Minimal V0는
  `Delta theta`를 항상 0으로 두어 learned orientation output이나 새 network를 만들지 않는다.
- 현행 M-norm release proxy는 이미 `sqrt(M_ref) L_ref`와 `sqrt(M_ref) V_ref`로 무차원화되어
  있으므로 보고서의 차원 오류 지적은 stale false positive로 반박했다. 대신
  `||u||_M=sqrt(u^T M u)` 정의를 명시해 해석 공백만 닫았다.
- MC1 topology distillation의 executable contract 공백은 수용했다. Anchor/candidate/feature,
  construction-level symmetric relation/layer logits, positive normalized area, training-only source-label,
  loss/decoder와 source-object grouped split을 version/hash로 고정했다. 다만 weighted FPS,
  geodesic-radius 또는 특정 architecture를 유일 해법으로 강제하지 않았고, 이 계약은 V0-R1/Gate A
  전에만 blocker다. Oracle V0-01--04와 Gate B early kill test는 막지 않는다.
- 기존 SI--nd 계약에서 빠졌던 areal density, stretch/bend stiffness, air density,
  wind/temporal/spatial derivative, typed bounds와 epsilon의 mapping을 보강했다.
- 통계 계약은 frame 독립표본화를 금지하고 sequence metric을 package로 aggregate한 뒤
  source-object outer/package nested paired cluster interval을 사용한다. Signed log-PSD 평균 대신
  RMS/L1 log-spectrum error를 사용하며, reference-construction failure는 method failure와 별도
  분모로 보고한다. Low-object-count 결과는 exploratory/descriptive로 표시하고 결과를 본 뒤 asset을
  소급 추가하지 않는다. Pareto pass에는 development-frozen practical margin과 interval lower bound를
  요구한다.
- Gaussian Swaying은 Gate 생존 뒤 코드/라이선스/적응 가능성을 확인할 direct-3DGS baseline 후보
  하나일 뿐 지금 canonical baseline으로 고정하지 않았다. 최종 title, novelty와 venue는 Gate A/B/C
  evidence 이후 V0-05에서만 freeze한다.

Runtime stage 5개, S0--S7 8개, milestone 7개와 deferred registry 8개를 유지했다. H/F/G,
KKT/Schur, same-frame corrector, confidence network, Teacher-FSI 또는 advanced solver를 core로
재승격하지 않았다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 재빌드했다.
- Sketch는 28쪽, checklist는 14쪽의 non-empty A4 PDF다.
- 두 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 Overfull box가 없음을 확인했다. Sketch bibliography의
  기존 Underfull 1건만 남았으며 내용·레이아웃 비차단이다.
- Bending/gauge, MC1 executable contract, clustered statistics/Pareto와 checklist typed artifact,
  S1, milestone 페이지를 raster inspection했고 clipping, 겹침, 깨진 수식과 `??`를 발견하지 못했다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다.
  두 bundle 모두 `unzip -t`를 통과했고 내부 entry hash가 working file과 일치한다.
- 최종 전역 감사에서 fitted-frame 정의, Gaussian scale/area reference와 epsilon type,
  clustered sampling hierarchy, Pareto floor, stable labels와 cross-document version field 사이에
  남은 blocker가 없음을 확인했다.

2026-08-18 최종 SHA-256:

- sketch TeX: `338298f15c9ac0f0ecc101c3c3d3d9af4527a30be30938aac8c4066d422011f4`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `7cd1a84e0630693f08f67f3432cb547b490338cc59da8db86d8698708dd02cd6`
- sketch bundle: `76bcc3c6b52477758bbb11cfdaa600384a90279829625a6f10aa64a9230abb84`
- checklist TeX: `a2a093080bfadd45058373407fea45229d3ea837f499048041e99aecc9694df4`
- checklist PDF: `09a73be711111e92a006e9a6c7500eab5b5317c545b48595452fcd63c6da134c`
- checklist bundle: `bceec0afbc97b8cc7cbc71283b30bf6128cd96b76b55f5498e5687781c4354f9`

이번 반영도 ideas 문서와 실행 계약 갱신이며 code/experiment 완료 상태를 생성하거나 승계하지 않았다.

## 2026-08-20 Codex review report 후속 반영

`wind3dgs_minimal_v0_codex_review_report_2026-08-20.md`를 current Minimal V0 canonical
sketch/checklist와 equation label 및 문구 단위로 대조했다. Review metadata의 source hash는 별도
붙여넣기 텍스트를 가리켜 canonical TeX hash와 직접 같지는 않았지만, 각 RV 항목이 인용한 식과
계약은 current source와 일치했다. 따라서 파일 전체를 기계적으로 덮어쓰지 않고 현행 source를
authority로 삼아 항목별로 수용·부분수용·반박했다.

- RV-01의 corotated tangent-plane bending 검증은 현행 수식이 이미 맞아 변경하지 않았다.
- RV-02는 수용했다. Attachment gauge eigenvector를 unit-normalize하고
  `L_att=g_att g_att^T`가 sign/scale에 무관한 rank-one orthogonal projector임을 명시했으며,
  norm, idempotence, trace와 replay fixture를 checklist에 동기화했다.
- RV-03의 SI--nd 및 M-norm release 식은 이미 차원이 맞아 유지했다. 물리적 mass 하나만 바꿔도
  release threshold가 불변이라는 처방은 반박하고, unit replay 또는 모든 owner quantity가 함께
  바뀌는 controlled similarity scaling에서만 invariance를 요구했다.
- RV-04는 용어만 교정했다. Stored `K`는 Global step과 basis가 공유하는 fixed-Hessian package
  operator이며 minimized nonlinear constraint energy의 exact rest tangent라는 주장을 하지 않는다.
  Solver와 basis construction은 바꾸지 않았다.
- RV-05는 calibrated absolute-area estimator를 만들지 않고
  `A_ref=L_ref^2`인 deterministic ownership convention으로 고정했다. 이는 물리적 절대 면적이나
  engineering mass 측정값이 아니며, area scale invariance는 관련 `M/C/K/load`와 typed floor,
  epsilon, clamp가 함께 similarity-scaled되는 조건으로 한정했다.
- RV-06은 split/test seal, training-static non-placeholder config, train/dev model·decoder 선택,
  test 전 full hash freeze 순으로 lifecycle을 닫았다. 이 미완료는 V0-R1/Gate A만 막고 Oracle
  V0-01--04 및 early Gate B/C를 막지 않는다.
- RV-07은 `y_rel=1`을 valid same-layer material relation, `y_layer=1`을 invalid cross-layer 또는
  shortcut으로 고정하고 decoder가 high layer probability를 reject하도록 polarity replay를 추가했다.
- RV-08은 rank/finite/orientation 검사 뒤에만 deterministic 3x3 SVD singular-value clamp를 적용하는
  `eq:minimal-affine-safe-map`을 추가했다. Reflection이나 rank loss를 sign repair 또는 clamp로 숨기지
  않고 기존 renderer-only proper-rigid fallback으로 보낸다. Surface/aero fail-fast는 유지한다.
- RV-09는 one-sided velocity-density PSD의 단위를 고정하고 기존 `(L0,T0)`로
  `P_hat=P T0/L0^2`, `epsilon_P_hat=epsilon_P T0/L0^2`, `f_hat=fT0`를 정의했다. 별도 PSD time
  scale은 만들지 않았고 SI ratio와 nd 결과가 같은지 replay한다.
- RV-10은 `J` singular value와 `G` eigenvalue에 unit-aware relative--absolute numerical-rank
  threshold를 추가하고 모든 inverse 이전에 검사하도록 했다. Threshold, unit branch와 CPU/GPU
  replay를 package hash에 포함했다.
- RV-11의 논문 재배치는 V0-05 이후 paper draft 작업이므로 canonical method/checklist에는
  반영하지 않았다.

Runtime stage 5개, S0--S7 8개, milestone 7개와 deferred registry 8개를 유지했다. 새 backend,
runtime stage, learned output, hard gate 또는 asset class를 추가하지 않았다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 재빌드했다.
- Sketch는 30쪽, checklist는 15쪽의 non-empty A4 PDF다.
- 두 최종 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 Overfull box가 모두 0임을 확인했다. Sketch bibliography의
  기존 Underfull 1건은 내용·레이아웃 비차단이다.
- Attempt/failure denominator, numerical-rank, affine-safe SVD, R1 lifecycle, PSD/statistics와
  checklist manifest/milestone 페이지를 raster inspection했고 clipping, overlap, 깨진 수식과 `??`가
  없음을 확인했다.
- Checklist의 모든 stable equation label이 sketch에 존재하고, source 내부 missing reference와
  duplicate label은 0이다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다.
  두 bundle 모두 `unzip -t`를 통과했고 내부 entry hash가 working file과 일치한다.
- 독립 교차 감사에서도 RV-02/04/05/06/07/08/09/10의 수학·차원·lifecycle과 두 문서의 계약에
  남은 blocker가 없음을 확인했다.

2026-08-20 최종 SHA-256:

- sketch TeX: `dedec9ecb7dcc70c7ac425096d8bee6372f4ca1e5cdfa23edadbdfe425473b7b`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `24f2664a7ba9fd15b5ff3ff074db8d27287d978b823ea212c505c7a4303c97ce`
- sketch bundle: `59864282a485755c22b44b00d05341ef7e23a429faeba6e57d4ad2a05a61ae08`
- checklist TeX: `ee1019991ad14119b5b70423e1af3a903ee1ee00c624eb8b1fbc8e92e1465ca8`
- checklist PDF: `9bdc1d46172788bb3097320eb6b1b2c6f80dcb3e58add9f8a027a8832747ada4`
- checklist bundle: `d40c99ee2a3f3a9420756a7cee09492ced4123cc01f10c5df5713ca5a9ea9e0c`

이번 반영도 ideas 문서와 실행 계약 갱신이며 code/experiment 완료 상태를 생성하거나 승계하지 않았다.

## 2026-08-20 Minimal V0 final verification 마감 반영

`review/wind3dgs_minimal_v0_final_verification_codex_report_2026-08-20.md`를
current canonical sketch/checklist와 다시 대조했다. Review metadata의 `source_sha256`은 canonical
TeX 파일명이 아니라 별도 입력 snapshot을 가리키므로 hash 자체를 freshness 증거로 사용하지 않았고,
인용된 식, label과 문구를 current source에 직접 대조했다. FV-01--FV-10의 수학·차원 계약에는
새 physics/solver blocker가 없었으며 다음의 국소적인 마감 항목만 반영했다.

- FV-11: selector 문맥의 `Oracle-selected`/`Oracle Local`을
  `privileged-teacher-ranked`로 축소했다. 이는 동일 atomic units와 coupled solver 위에서 privileged
  future-window teacher가 unit별 순위를 제공하는 scope-control diagnostic이지, exact best-subset
  oracle 또는 formal upper bound가 아니다. `Oracle scaffold`, source fixture/package와 deferred-F의
  별도 correction 문맥은 보존했다.
- FV-12: 기존 area decode label을 유지하면서
  `zeta_i^a=g_{theta,a}(h_i)`인 shared scalar head를 명시했다. 입력은 각 node embedding `h_i`만,
  출력은 stacked `|V| x 1`이며 global pooling이나 새 message-passing stage를 추가하지 않는다.
  Architecture/output field ID와 hash는 기존 `nu_arch` metadata가 소유한다.
- FV-13: temporal frequency만 `f_PSD`, `hat f_PSD`로 분리했다. Review가 제안한 bare `nu`는 이미
  version-ID 기호에 쓰이므로 채택하지 않았고, 물리 force의 `f`, `hat f` typed map은 그대로 유지했다.
- PSD L1과 RMS는 모두 보고하되 development에서 정확히 하나를 primary로 사전 등록하고 다른 하나를
  mandatory secondary로 고정했다. 결과를 본 뒤 primary를 바꾸지 않는다.
- Checklist의 FV-10 lifecycle에는 decoder schema, training seed/data-label hash, selected model weight와
  source/label/data/config/decoder/split을 포함한 test-open 전 immutable full hash를 명시했다.
- S2 fixture 문구는 실제 fitted rest/predictor tangent frame과 stored rest displacement `d_ia^0`를
  사용하도록 정리했다. Exact `w_bend` 검사는 `delta J_i=0`인 unit fixture에만 요구한다.

FV-03 review acceptance의 mass-only scaling invariance는 채택하지 않았다. Release proxy invariance는
unit replay 또는 `M/C/K/load`와 typed epsilon/floor가 함께 변하는 controlled similarity scaling에서만
성립한다. Runtime 5단계, S0--S7 8개, milestone 7개, deferred registry 8개를 그대로 유지했고 새
backend, runtime stage, learned output, gate 또는 asset class를 만들지 않았다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 실제 재빌드했다.
- Sketch는 30쪽, checklist는 16쪽의 non-empty A4 PDF이며 각 PDF/log가 source보다 최신이다.
- 두 최종 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning, Overfull box가 모두 0이다. Sketch bibliography의 기존
  Underfull 1건만 남았고 내용·레이아웃 비차단이다.
- Gate B의 긴 teacher-ranked 비교식을 여러 줄로 나눠 overfull을 제거했다. Gate B, area head,
  PSD typed metric과 checklist artifact/Gate 페이지를 raster inspection했고 clipping, overlap,
  깨진 수식과 `??`가 없음을 확인했다.
- Sketch label 47개는 unique하고 source 내부 missing reference 및 checklist stable-label 누락은 0이다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다. 두 bundle 모두
  `unzip -t`를 통과했고 각 내부 entry SHA-256이 working file과 일치한다.
- 독립 교차 감사에서도 FV-11/12/13, FV-10 freeze lifecycle와 S2 wording에 남은 blocker가 없음을
  확인했다.

2026-08-20 final verification SHA-256:

- sketch TeX: `1e117d292a107c34f901b26becfc4e4a919c3d452aaad5586534ce708e2a8888`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `3ddadd45da50fa0cf7392ce36cd498c77a2263f7d257879b5088d737c689d53f`
- sketch bundle: `740fbcfb2799b667953cbefbad1b393d06760b0d55ed632cce2d06821c4d55cf`
- checklist TeX: `d402c1c4744a0fe2aabb6a72fbbaab36f97b7324c98c99e4a24ea01beaf74fdd`
- checklist PDF: `1da83afb80e692dd9ee6b812fd60e615f66a52bbf44bcad8c6c384d08787562b`
- checklist bundle: `9d2214d60408924d8d951db669965ddb3de1a1a8011b51906990fff0bc8e43ee`

이번 반영도 ideas 문서와 실행 계약 갱신이며 code/experiment 완료 상태를 생성하거나 승계하지 않았다.

## 2026-08-20 Minimal V0 feedback의 milestone-local 계약 반영

`review/wind3dgs_minimal_v0_feedback_codex_report_2026-08-20.md`를 직전 canonical source와
scope-control 종료 기준으로 대조했다. 보고서의 `source_lines=1938`과 인용 label/문구는 current
snapshot과 일치했고, `source_sha256=b1d4afa...`는 직전 canonical TeX를 CRLF로 변환한 byte hash와
정확히 같았다. 따라서 stale review는 아니지만, 새 method P0와 owner-milestone 구현 선택을 구분해
다음 최소 계약만 반영했다.

- FB-01은 V0-01-local 계약으로 수용했다. Anchor scalar mass에서
  `M=blkdiag_i(m_i I_3)`를 만드는 `mass_matrix_type=lumped_anchor_v1`을 고정하고, stable anchor
  ordering의 ordered `m_i` 또는 equivalent repeated diagonal과 hard-attachment free map만 hash한다.
  Dense `M` 저장, consistent/learned mass 또는 새 basis algorithm은 추가하지 않았다.
- FB-02는 V0-02-local 계약으로 수용했다. Analytic raw aero를 먼저 계산하고 finite 검사 뒤
  `f_max>0` Euclidean radial hard clamp를 정확히 한 번 적용한다. Bounded force만 direct projection이
  소비한다. Preregistered required anchor--frame 집합 `I_f`를 고정 분모로 쓰고 finite clamp event만
  `r_f,clip` numerator에 센다. Non-finite raw force는 분모에서 삭제하지 않는 별도 method failure다.
  새 threshold/gate를 만들지 않고 기존 generic clamp-rate cap만 사용한다.
- FB-03의 최종 `nu_decode`를 training 전에 요구하는 처방은 반박했다. `PRE_TRAIN`은 decoder schema와
  threshold/cap candidate space를 고정하고, final numerical threshold/cap과 model은 development에서만
  선택한다. `PRE_TEST`에서 dev-selected final `nu_decode`, model weight와 full immutable hash를 처음
  완결한다. 이는 test-aware retuning을 막으면서 dev-selection을 보존한다.
- FB-04는 V0-03/Gate-B-local offline replay로만 수용했다. `teacher_counterfactual_v1`은 full serialized
  `RuntimeState`와 모든 per-unit dwell/fade/release timer/counter를 candidate마다 deep-copy한다.
  Minimum-dwell mandatory set을 base request로, candidate 하나를 highest optional priority로 넣은 request를
  비교하며 같은 wind/reference/solver/precision/seed와 기존 capacity-`K_A` admission policy를 사용한다.
  Candidate를 강제 삽입하거나 state를 reset하지 않고, horizon 안의 recursive re-ranking/subset search도
  하지 않는다. 첫 frame 미입장을 score 0으로 단정하지 않고 전체 rollout trace가 동일할 때만 0으로 둔다.
  Gate-B manifest와 selector report가 protocol/metric/tie-break 및 initial-state/wind/reference/config hash를
  소유한다.

Runtime 5단계, S0--S7 8개, milestone 7개, deferred registry 8개를 유지했다. 새 runtime state는 만들지
않고 기존 state field를 typed artifact에 명시했으며, solver/network/gate/asset class도 추가하지 않았다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 실제 재빌드했다.
- Sketch는 31쪽, checklist는 17쪽의 non-empty A4 PDF이며 각 log/PDF가 source보다 최신이다.
- 두 최종 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning과 Overfull box가 모두 0이다. Sketch bibliography의 기존
  Underfull 1건만 남았고 내용·레이아웃 비차단이다.
- Lumped mass, raw/bounded aero와 clamp-rate, Gate-B counterfactual, checklist typed artifact/S3/S4/Gate-B
  페이지를 raster inspection했고 clipping, overlap, 깨진 수식과 `??`를 발견하지 못했다.
- Sketch label 47개는 unique하고 source 내부 missing reference 및 checklist stable-label 누락은 0이다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다. 두 bundle 모두
  `unzip -t`를 통과했고 각 내부 entry hash가 working file과 일치한다.
- 독립 교차 감사에서 mass/xyz repeat, force-clamp 분모와 기존 cap, PRE_TRAIN--PRE_TEST lifecycle,
  mandatory-base/Decay timer/full-horizon teacher replay와 5/8/7/8 범위에 남은 blocker가 없음을 확인했다.

2026-08-20 feedback contract SHA-256:

- sketch TeX: `efbb0b14bc456fdce56be705fa34722e7962dcab866c3c2236d26e9ecfb797ec`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `1c8c5f3e18ca21d2cc181ab87313edb2d794d9ee6c23350a0623e9bba6819934`
- sketch bundle: `e248df8e14b99bd04712a13310cc2e424647bcb18e15d90877178cb67e88923e`
- checklist TeX: `f1944b7050ccd10866562c5c65fc943803a6b5c8569c4c1db53aecbe8c9e3e9f`
- checklist PDF: `08563f7d9806b34605bd180b6da3fb3131200bf2593d81d11280a0b52f223aa8`
- checklist bundle: `77bc5b02079fd8ac0584e2cc823132a8d137eeb563efe5a83ca40e5fd5ad6de4`

이번 반영도 ideas 문서와 milestone-local 실행 계약 갱신이며 code/experiment 완료 상태를 생성하거나
승계하지 않았다.

## 2026-08-21 구현 착수 가능성 리뷰의 milestone-local 계약 반영

`review/minimal_v0_review_report.tex`를 canonical sketch/checklist와 대조했다. 보고서가 기록한 source
snapshot은 직전 canonical sketch의 CRLF 변환본과 byte hash가 일치해 fresh했다. 이 리뷰의 관점은
물리 정확도나 신규 method 추가가 아니라, 두 구현이 같은 trace와 same-anchor p95를 재현하도록
implementation handoff의 ambiguity를 닫는 것이다. 보고서의 `IMPLEMENT`, P0 없음 판정은 method-spec
착수 가능성에 한정해 수용했고, code/data readiness 또는 연구 가설 입증으로 확대하지 않았다.

다섯 P1은 owning milestone 안에서만 다음과 같이 최소 반영했다.

- Atomic Local group은 stable member/unit/column/slot ordering과 exact serialized retained group basis를
  reduced-package hash가 소유한다. Merge/prune policy는 유지하고 unconditional member-basis concatenation은
  강제하지 않았다. `K_A`와 exact package identity는 selection config 및 run manifest/hash가 소유한다.
- Decay release의 energy, displacement, velocity, hold thresholds는 모두 finite positive이며 frame-count,
  `K_A`, `K_D`는 positive integer라는 validator를 고정했다. 기존 strict-inequality release 의미는 유지했다.
- 한 frame의 순서를 pre-solve Active/Decay membership 결정, 단일 provisional coupled solve, post-solve
  release predicate/hold-counter 갱신, frame-end released-slot zeroing, 다음 frame membership 적용으로
  고정했다. 같은 frame 재풀이, 새 runtime state 또는 두 번째 solve는 추가하지 않았다.
- Current-surface area는 raw ratio와 operating-envelope clamp를 통과한 `a_safe`를 구분하고, aero consumer는
  이전 frame의 safe area만 사용한다. 기존 generic area-bound/rate-cap을 재사용했으며 새 gate 또는 별도
  index system을 만들지 않았다.
- Evaluation floors `epsilon_v`, `epsilon_x`는 finite positive이고 각각 energy와 squared-length unit을 가지며,
  SI--nd inverse map과 `M_eval` convention을 evaluation hash가 소유한다. Zero-motion reference에서도 metric
  denominator가 finite하도록 고정했다.

정확한 `oracle_flat_strip_3x7` fixture는 유용한 code-side 예시지만 canonical architecture 요구로 넣지
않았다. Runtime 5단계, S0--S7 8개, milestone 7개, deferred registry 8개와 기존 gate 수를 유지했고,
solver/network/state class/asset class를 추가하지 않았다.

### 검증

- 두 canonical TeX를
  `latexmk -g -xelatex -interaction=nonstopmode -halt-on-error`로 실제 재빌드했다.
- Sketch는 32쪽, checklist는 17쪽의 non-empty A4 PDF이며 각 PDF/log가 source보다 최신이다.
- 두 최종 log에서 LaTeX/package error, undefined citation/reference/control sequence,
  multiply-defined label, rerun warning, Overfull box와 amsmath warning이 모두 0이다.
- Sketch의 변경 밀집 페이지 14, 15, 18, 26과 checklist 페이지 5, 7, 11, 13을 raster inspection했고,
  clipping, overlap, 깨진 수식과 `??`를 발견하지 못했다.
- Sketch label 47개는 unique하고 source 내부 missing reference 및 checklist stable-label 누락은 0이다.
- Sketch bundle은 TeX/Bib/PDF 정확히 3개, checklist bundle은 TeX/PDF 정확히 2개다. 두 bundle 모두
  `unzip -t`를 통과했고 모든 내부 entry SHA-256이 working file과 일치한다.
- 독립 교차 감사에서 atomic/hash ownership, positive release threshold, causal frame order, safe-area
  consumer와 evaluation-floor unit/hash에 남은 blocker가 없음을 확인했다.

2026-08-21 implementation-readiness contract SHA-256:

- sketch TeX: `e54f6610be3f4a142b351069ddfb29492bfef2bc01e4ec8198afe537fe1b3a3e`
- bibliography: `067ef195d7d3c7042aec4d5db51ce70c4b85bb0bce1ae9e422f9aa5c35f8b99d`
- sketch PDF: `3a100e07a286b6e3344f1282dbc6b7192441919cca967db9e89b62c99a01850c`
- sketch bundle: `fd2d2c2d0ecd8b4c67c55823cdb3dc216ab7e87ec71e97ba06f94c874ae24c36`
- checklist TeX: `07e702f04bb290310216ef893a34a902ad1971244c763827958cbbde7b538458`
- checklist PDF: `64782af4ab02d1303d1b22b2b0d545699203a063dd9dbae308218ca10d5a67d3`
- checklist bundle: `009f00ae7208b4d90268372948decf50d543c2bb1ac82a15d1fc954c6e775368`

이번 변경은 ideas 문서와 milestone-local 실행 계약만 갱신한다. Code/experiment 구현, 실행 결과 또는
완료 상태를 생성하거나 승계하지 않았다.

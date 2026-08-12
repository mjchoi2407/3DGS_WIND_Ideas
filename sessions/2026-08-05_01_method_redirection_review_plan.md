# 2026-08-05 01 method redirection 검토와 진행 계획

## 배경

사용자가 `ideas/3dgs_wind_response_idea_sketch_2026-08-05_bundle.zip`에 수정된 연구 아이디어를 제공하고, 내용을 확인한 뒤 진행 계획을 세워 달라고 요청했다.

이번 번들은 기존 `persistent runtime mesh proxy` 설계를 보완한 문서가 아니라, source mesh를 training-only privileged teacher로 사용하고 target inference에서는 static 3DGS만 사용하는 구조로 main method를 교체한 문서다.

공통 Startup Protocol에 적힌 다음 파일은 현재 workspace와 `/home/choi/projects/2026_paper_work` 아래에서 찾지 못했다.

- `../RESEARCH_PROJECT_GUIDE.md`
- `../templates/research_project/TEMPLATE_MANIFEST.md`

따라서 root/ideas `AGENTS.md`, 현재 ideas 기록, 최근 code/experiments/session 기록을 기준으로 검토했다.

## 번들 무결성과 빌드 확인

- ZIP SHA-256:
  - `5b1497691957c41d8f83e53b3212d3ca0a2f0410a4e4f49263de299d063e2b2d`
- 구성:
  - `BUILD_README.md`
  - `3dgs_mesh_privileged_local_global_wind_response_2026-08-05.tex`
  - `3dgs_mesh_privileged_local_global_wind_response_2026-08-05.pdf`
  - `refs_3dgs_wind_response.bib`
- `unzip -t`: 모든 entry 통과
- 제공 PDF: 66 pages
- 임시 폴더에서 `latexmk -xelatex -interaction=nonstopmode -halt-on-error ...tex` 재빌드 성공
- 최종 빌드에는 `microtype` footnote patch warning만 남았고 citation/undefined-reference error는 없었다.

## 핵심 방향 전환

기존 구조:

```text
target 3DGS -> extracted runtime mesh proxy -> runtime mesh solver
            -> triangle binding -> dynamic Gaussians
```

새 구조:

```text
training only:
source mesh -> multi-view RGB -> independent static GS variants
source mesh -> shell teacher simulation -> topology/trajectory/F supervision

target setup/runtime:
static 3DGS -> Gaussian anchors -> inferred material graph
            -> overlapping core-halo patches
            -> analytic wind prior + global response + shared local residual
            -> second-order rollout
            -> continuous RBF/MLS mean-covariance transport -> GS rendering
```

정확한 claim은 `mesh-supervised and topology-distilled, but target-mesh-free at inference`다. Object input을 `RGB-only`라고 부르는 것은 부정확하다. Target은 calibrated multi-view RGB로 offline reconstruction된 static 3DGS이고, runtime input은 `static 3DGS + wind + material + constraint + camera`다.

권장 main contribution은 다음 세 항으로 좁힌다.

1. Source-mesh geodesic topology를 independently reconstructed static GS anchor graph에 distill하는 material scaffold
2. Explicit wind/material/pin condition을 받는 coarse global + shared local response operator
3. Partition-of-unity field와 Jacobian으로 Gaussian mean/covariance를 갱신하는 continuous transport

## 기존 기록과의 충돌

현재 `idea_sketch.tex`, `changelog.md`, `implementation_checklist.md`는 새 설계와 동기화되지 않았다.

- Runtime mesh extraction과 mesh solver는 main critical path에서 제거해야 한다.
- 기존 M02 triangle/barycentric binding은 폐기하지 않고 training-only teacher transfer 검증으로 재배치한다.
- 기존 M03 procedural wind deformation은 wind control schema와 synthetic fixture로만 재사용하며 teacher physics로 사용하지 않는다.
- 기존 fixed material-space load sample 및 dense-to-proxy force projection 설계는 새 문서에 없다. 새 runtime은 anchor에서 wind와 analytic load를 직접 계산한다.
- 2026-07-07 boundary-continuous response map 결정은 새 문서에 없다. Initial thin-shell paper의 MVP에서는 보류하되, spatial material/response authoring extension으로 기록을 보존하는 편이 적합하다.
- 기존 기록은 learned SH residual을 필수로 두지만, 새 문서는 analytic SH rotation을 기본, residual network를 optional로 둔다. 새 방향을 따르면 residual은 V4에서 metric으로 필요성이 입증될 때만 승격한다.
- M04 mesh-extraction 결과와 스크립트는 삭제하거나 재편하지 않는다. Pivot의 실패 근거와 비교 baseline으로 보존하고 새 실험은 별도 폴더에 격리한다.

## 구현 전 P0 명세 수정

Network나 대규모 dataset을 만들기 전에 다음 항목을 결정·수정해야 한다.

1. 수식 오타
   - tex line 2683의 `\ho_sh_r`를 `\rho_s h_r`로 수정한다.
2. Anchor area/mass 보존
   - 현재 `sum_i chi_ki=1`은 local 평균에 가깝고 anchor 담당 총면적/질량을 보존하지 않는다.
   - mesh vertex/face mass를 anchors에 partition하여 `sum_k A_k=A_mesh`, `sum_k m_k=m_mesh`를 검사한다.
3. Reliability weight 소거
   - `q_j^G`가 Gaussian별 anchor softmax에서 공통 인자라 normalize할 때 소거된다.
   - anchor aggregation까지 reliability가 실제 weight로 남도록 식을 다시 정의한다.
4. Dimensionless response scale
   - current relative-wind RMS `U_r^t`만으로 output을 `U^2/L` 복원하면 zero-wind turning point에서 elastic restoring acceleration이 0으로 붕괴할 수 있다.
   - fixed reference speed 또는 structural time/acceleration scale을 추가한다.
5. Shell deformation gradient
   - 평면 anchor neighborhood의 `B_k^0`는 rank 2이므로 `epsilon I`만으로 normal inverse-transpose를 계산하지 않는다.
   - tangent frame + explicit normal axis, 2D shell gradient, 또는 local Procrustes/polar rotation을 검증한다.
6. RBF linear precision과 reflection correction
   - normalized RBF는 일반 rigid/affine motion을 정확히 재현하지 않는다. V0 rigid test 실패 시 MLS/affine correction을 필수로 승격한다.
   - `det(J)<0` correction에는 실제 sign matrix를 식에 명시한다.
7. Constraint supervision
   - mesh pin field를 anchor `c_k`와 constraint reaction target에 투영하는 정의를 추가한다.
8. Patch frame degeneracy와 force assembly
   - PCA의 `lambda_1 ~= lambda_2`일 때 tangent orientation을 안정화한다.
   - patch별 antisymmetric force가 final blend 뒤에도 global momentum consistency를 유지하는지 별도 검사한다.
9. Teacher/runtime exposure 일치
   - V4 exposure를 켤 때 teacher에도 같은 attenuation을 적용하거나 별도 fine-tuning target으로 분리한다.

## Novelty gate

번들의 related work는 방향을 설명하기에는 유용하지만, 현재 날짜 기준 가까운 연구가 빠져 있다.

- GausSim, ICCV 2025: hierarchical coarse-to-fine Gaussian neural simulator와 momentum/physics constraints
  - https://openaccess.thecvf.com/content/ICCV2025/html/Shao_GausSim_Foreseeing_Reality_by_Gaussian_Simulator_for_Elastic_Objects_ICCV_2025_paper.html
- MaGS, ICCV 2025: mesh-adsorbed Gaussian reconstruction/simulation
  - https://openaccess.thecvf.com/content/ICCV2025/html/Ma_MaGS_Reconstructing_and_Simulating_Dynamic_3D_Objects_with_Mesh-adsorbed_Gaussian_ICCV_2025_paper.html
- Neural Gaussian Force Field, ICLR 2026: fast learned Gaussian physical dynamics from multi-view input
  - https://arxiv.org/abs/2602.00148
- Learning a Particle Dynamics Model with Real-world Videos, CVPR 2026 Findings: dense Gaussian particle dynamics with rendering supervision
  - https://arxiv.org/abs/2605.23845
- Gaussian Swaying, DynamicTree, DiffWind, ParticleGS도 기존 비교와 함께 유지한다.

Local/global hierarchy, neural Gaussian dynamics, momentum loss 자체를 novelty로 주장하기는 어렵다. 구현 전 1~2 page claim matrix를 만들고, main novelty를 `topology-distilled GS material scaffold + explicit wind/material/pin control + independently reconstructed static GS + target-mesh-free runtime + covariance transport`로 방어할 수 있는지 확인한다.

## Falsification-first 진행 계획

### Gate 0: 문서·데이터 계약·환경 고정

목표:

- 새 `.tex`를 authoritative spec으로 정리하고 2026-08-05 pivot을 changelog에 반영한다.
- 기존 checklist를 V0--V4 구조로 교체한다.
- `asset_manifest`, `teacher_sequence`, `GS_variant`, `student_input` schema를 먼저 고정한다.
- Student input allowlist를 두어 mesh vertex/face, triangle id, barycentric coordinate, geodesic label이 runtime batch로 새지 않게 한다.

환경:

- 현재 `blender` 실행 파일이 없다. Source multi-view renderer는 Blender 설치, `sugar` env의 `nvdiffrast`, 또는 작은 자체 rasterizer 중 1-day spike로 선택한다.
- Dynamic GS renderer/reconstruction은 GTX 1080 Ti에서 이미 검증된 GOF/graphdeco 경로를 우선 사용한다.
- root `.venv`의 `gsplat 1.5.3`은 CC 6.1에서 적합하지 않다.
- renderer/data generation env와 dynamics-training env를 분리하고 파일 schema로 연결한다.

Stop condition:

- coordinate/camera contract를 unit cube와 axis triad로 재현하지 못하면 이후 작업을 진행하지 않는다.

### Gate 1: V0 teacher-to-GS transfer closed loop

첫 입력:

- 기존 M02 `10x10` 또는 `30x30` textured cloth 1개
- identity, translation, rigid rotation, known bend/twist trajectory
- 40--80 calibrated views
- GOF reconstruction 1 seed부터 시작하고 통과 뒤 두 번째 independent variant를 추가한다.

구현:

- source mesh와 reconstructed GS 사이 closest-point/barycentric/offset/confidence correspondence
- mean-only, rigid-frame covariance, full-F 또는 affine-corrected MLS transport 비교
- held-out camera에서 mesh teacher와 dynamic GS render 비교

권장 hard gate:

- valid mapping >= 99%
- identity/translation/rigid mean·covariance relative error <= `1e-4`
- covariance SPD 100%
- held-out silhouette IoU >= 0.95
- covariance-aware transport가 mean-only보다 hole/flicker를 의미 있게 줄임

이 단계가 실패하면 XPBD나 GNN을 구현하지 않는다.

### Gate 1.5: Teacher shell solver 검증

- Regular cloth에서 stretch/shear/bend, lumped mass, pin, damping, one-way aerodynamic load, fixed `dt/substeps`를 구현하거나 검증된 solver를 연결한다.
- `x, v, a, n, F, external force, constraint reaction, units, solver settings`를 export한다.
- 초기에는 정확한 `E, nu` 역복원보다 normalized stiffness/compliance preset을 사용한다.

권장 gate:

- zero-wind rest drift < `1e-5 L`
- 동일 seed/settings deterministic rerun
- damping 시 energy 감소
- timestep/substep refinement trajectory RMS < `2% L`
- moderate regime에서 NaN, face flip, 과도한 strain 없음

### Gate 2: Static-GS topology 식별성 probe

Dynamics 대규모 학습 전에 flat sheet, U-fold, close-disconnected double sheet 등 Euclidean shortcut을 의도적으로 만드는 3--5 assets, 각 2 GS variants를 사용한다.

비교:

- Euclidean k-NN
- normal-aware k-NN
- 작은 topology MLP
- ground-truth mesh graph upper bound

권장 gate:

- candidate positive recall >= 98%
- learned topology F1 >= 0.90
- hard-negative FPR < 5%, Euclidean 대비 50% 이상 감소
- anchor가 close layers 중간으로 collapse하지 않음

실패하면 object family를 isolated single layer로 줄이거나 anisotropic anchor assignment/user topology correction을 채택한다. Static GS evidence가 동일한 two-layer case는 원리적으로 식별 불가능하므로 arbitrary claim을 하지 않는다.

### Gate 3: V1 oracle-topology asset-specific dynamics

- 1 cloth, `K=128--256` anchors, 8--32 wind/material/pin sequences
- Ground-truth mesh topology를 oracle graph로 제공
- analytic aero-only와 단순 monolithic graph GNN부터 비교
- one-step training 뒤 2--4 second autoregressive rollout

권장 gate:

- analytic-only보다 trajectory/amplitude/frequency/phase 개선
- 2--4 s normalized position error < `5% L`의 초기 목표
- exact pin, NaN/divergence 0

### Gate 4: V2 oracle-topology local/global generalization

- Feasibility split: 최소 8 train / 2 validation / 3 held-out source shapes
- shape, aspect ratio, material, pin, wind axes를 분리하고 object-disjoint split
- local-only, global-only, local+global 비교

권장 gate:

- local+global이 best single branch보다 unseen-object 및 changed-pin trajectory/phase에서 일관되게 10--15% 이상 개선

실패하면 두 branch contribution을 단순화한다. Patch당 constant global acceleration이 부족하면 node-level coarse basis/output을 검토한다.

### Gate 5: V3 topology-distilled inference

- Multiple independent GS variants, hard negatives, area/mass latent, inferred graph patching
- learned graph, Euclidean graph, oracle graph의 topology와 end-to-end dynamics gap을 모두 보고한다.

권장 gate:

- inferred-graph dynamics가 oracle metric의 10--15% 이내
- Euclidean graph보다 topology와 dynamics 양쪽에서 유의하게 개선

이때부터만 target mesh-free setup/runtime main claim을 사용할 수 있다.

### Gate 6: V4 CG-quality system

- Covariance-aware transport를 먼저 고정한다.
- Opacity exposure, polar SH rotation, optional learned SH residual은 마지막에 추가한다.
- Real captured GS, interactive wind/material/pin controls, runtime/memory profile을 수행한다.
- `N_G={20k,50k,100k}`, `K={128,256,512}`, `K_T=4` 같은 scaling sweep을 사용한다.
- GTX 1080 Ti에서는 real-time을 선결 claim으로 두지 않고 quality/latency Pareto를 먼저 보고한다.

## 기존 자산 재사용

- M01:
  - PLY mean/scale/quaternion/opacity/SH parser와 camera loader 재사용
  - synthetic leaf는 independent reconstruction이 아니므로 V0 final evidence로는 부족
- M02:
  - cloth OBJ/UV/topology/pin fixture, barycentric/local-frame/covariance tests 재사용
  - 현재 covariance path는 rotation 중심이라 full stretch/Jacobian transport로 확장 필요
- M03:
  - wind parameter schema와 UI/preview만 재사용
  - hand-authored displacement를 teacher physics로 사용하지 않음
- M04:
  - GOF train/render wrapper와 GPU 환경 재사용
  - mesh extraction 결과는 pivot evidence/baseline으로 보존
  - Mip-NeRF 360/CO3D real scenes는 source mesh correspondence가 없어 V4 qualitative target으로만 사용

## 저장·계산 비용 원칙

- V0는 1 source, 1 GS seed, 저해상도, 짧은 iteration으로 시작한다.
- Gate 통과 후에만 second seed와 asset 수를 늘린다.
- 모든 per-Gaussian trajectory를 저장하지 않고 mesh trajectory + static correspondence를 저장하여 target을 chunk/on-demand 생성한다.
- 대표 run만 checkpoint를 보존하고 GS variant는 final PLY, config, manifest, metrics 위주로 남긴다.
- Patch count를 독립 data count로 보고하지 않고 source shape/material/pin/wind/GS variant 수를 각각 보고한다.

## 이번 작업에서 바꾸지 않은 것

- 기존 `idea_sketch.tex`, `changelog.md`, `implementation_checklist.md`는 수정하지 않았다.
- ZIP 내용은 workspace에 추출하거나 덮어쓰지 않고 `/tmp`에서 읽고 빌드했다.
- 기존 code/experiments 파일과 대용량 결과를 변경하지 않았다.

## 변경 파일

- 추가: `ideas/sessions/2026-08-05_01_method_redirection_review_plan.md`

## 다음 작업

사용자가 계획을 승인하면 다음 순서로 진행한다.

1. 새 번들의 `.tex/.bib/.pdf`를 기존 dirty worktree를 보존하면서 authoritative ideas 문서로 반영한다.
2. `changelog.md`에 2026-08-05 method redirection과 기존 결정의 supersession 관계를 기록한다.
3. `implementation_checklist.md`를 V0--V4/gate 구조로 갱신한다.
4. `code/wind3dgs/m05_teacher_transfer/`와 `experiments/M05_teacher_transfer_sanity/`에서 Gate 0/1을 시작한다.

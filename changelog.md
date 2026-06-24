# Research Changelog

## 2026-05-18

- Decision: Updated the idea sketch and implementation checklist to keep SH residual correction as a required runtime module while reducing advanced anchor/load scoring to a refinement path.
- Reason: The system becomes too complex if every component is treated as a primary contribution, but SH residual correction is still needed to avoid per-frame SH optimization during runtime rendering.
- Evidence: The current core runtime stack is `proxy deformation -> Gaussian transport -> lightweight SH residual correction -> 3DGS rendering`; `view-masked gated floater` and `projection-risk-aware load sampling` are useful robustness mechanisms but should not block the first solver/rendering loop.
- Next: Implement M4 with fixed simulation anchors and simple fixed load samples first, then preserve M9 as the required runtime appearance milestone before final paper evidence. Keep `ideas/anchor_method_revision_notes_2026-05-11_v3.tex` as the detailed reference for advanced anchor/load scoring.

## 2026-05-11

- Decision: Added the v3 simulation-anchor/load-sample design to the idea sketch in compressed form, with `ideas/anchor_method_revision_notes_2026-05-11_v3.tex` kept as the detailed reference.
- Reason: Raw Gaussian count should not be treated as physical importance, isolation should only act through a view-masked gated floater likelihood, and frame-varying wind exposure should not cause frame-varying anchor selection.
- Evidence: The v3 anchor notes separate persistent simulation anchors from fixed material-space load samples, define reliability-weighted Gaussian transport demand, and use dense-force-supervised projection risk for one-time load-sample refinement.
- Next: Implement the MVP anchor/load sampler under `code/wind3dgs/`, then validate with ablations for raw density, direct isolation reliability, fixed load samples, and projection-risk refinement.

- Decision: Reframed constraint and material setup as a profile-conditioned, semi-automatic, user-editable physical/material field initialization layer.
- Reason: A universal automatic anchor detector is ill-posed across assets such as rubber-like Stanford Bunny, folded paper airplane, kite, cloth, and leaf. The more defensible framing is to initialize editable attachment, material, feature/structure, wind-response, and handle fields from object profiles, geometry cues, Gaussian support, and user edits.
- Evidence: The M4 design discussion separated physical attachment constraints from reduced simulation control nodes, and clarified that creases, ribs, spines, and frames are structure or stiffness constraints rather than fixed anchors.
- Next: Define reduced simulation control node sampling separately from controllable physical/material fields.

## 2026-04-30

- Created the initial repository research structure.
- Set the first paper direction around controllable, lightweight, real-time wind deformation for 3DGS assets.
- Marked DiffWind as a key related work and novelty boundary.

## 2026-05-03

- Decision: Refined the method toward `3DGS input -> lightweight simulation mesh proxy -> Gaussian binding -> wind deformation -> SH residual correction`.
- Reason: Vanilla 3DGS lacks topology and stable surface correspondence for deformable-body wind simulation, while direct Gaussian-patch aerodynamics overlaps with Gaussian Swaying.
- Evidence: Related work suggests that DiffWind uses 3DGS-derived particles with MPM, Gaussian Swaying uses Gaussian surface patches for aerodynamic force computation, and SuGaR/GOF focus on 3DGS surface or mesh extraction.
- Next: Prototype mesh-proxy binding and compare canonical SH, analytic SH/frame correction, learned SH residuals, and per-frame SH optimization.

## 2026-05-03 Formula Update

- Decision: Imported applicable equations from `paper/wind_deformable_3dgs_algorithm_v8.pdf` into the idea sketch.
- Applied directly: local wind field, wind-axis transmittance, exposure sampling, effective wind, force-space load prediction, scene-calibrated parameter field, parameter regularization, and conservation-preserving force projection.
- Adapted: ED-Graph/DQS anchors were rewritten as simulation mesh vertices, triangle barycentric Gaussian binding, and mesh-local frame transport.
- Not adopted as primary claims: direct deformation regression, pure Gaussian-patch aerodynamics, arbitrary unseen-asset generalization, and full CFD-quality wind simulation.
- Next: Use the new mathematical formulation to define the first mesh-proxy binding prototype.

## 2026-05-03 Framing Update

- Decision: Keep the current technical structure, but frame the paper around topology-aware wind-deformable 3DGS representation rather than AI-based SH coefficient prediction.
- Reason: SH residual prediction is valuable as an appearance correction module, but the stronger TVCG-facing contribution is the full 3DGS-to-simulation-proxy-to-Gaussian-rendering formulation.
- Next: Put mesh-proxy representation and force-space wind deformation first in future abstract, contribution, and title drafts.

## 2026-05-03 Necessity and Adaptation Update

- Decision: Added a defense against the "known components combination" critique.
- Reason: The method uses existing families of techniques, so novelty must be argued through the necessary representation bridge and domain-specific adaptation.
- Adaptation novelty candidates: Gaussian attribute transport through mesh-local binding, simulation-oriented proxy extraction, Gaussian-opacity wind exposure, conservation-preserving dense-to-proxy force projection, and deformation-aware SH residual transport.
- Next: Prioritize ablations that prove each bridge component is necessary.

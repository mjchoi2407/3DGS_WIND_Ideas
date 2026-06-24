# 2026-06-11 pipeline input detail

## Context

The user asked to strengthen the `Pipeline-Level Input and Output` input section in `ideas/3dgs_wind_response_pipeline_2026-05-12.tex`, because the previous table only named each input term without explaining what the term means or how its value is obtained.
The follow-up requests asked to apply the same explanation style to Module 1 and then to the remaining chapters.

## Decisions

- Keep the formal pipeline input as a trained 3DGS asset:
  `G^0=\{g_j^0\}_{j=1}^{N_G}` with `g_j^0=(\mu_j^0,R_j^0,s_j^0,\alpha_j^0,c_j^0)`.
- Explain each Gaussian attribute as a canonical rendering state, not just a symbol name.
- Document the practical Inria-style 3DGS PLY conversion rules:
  position fields for centers, normalized quaternion to rotation matrix, exponential log-scale conversion, sigmoid opacity conversion, and SH coefficient loading from `f_dc_*` / `f_rest_*`.
- Add derived preprocessing quantities for covariance and a Gaussian normal candidate.
- Expand additional inputs for wind field, user constraints, and camera sequence.
- Add the same interpretation to `Module 1: 3DGS Preprocessing`: `G^0` is the whole trained 3DGS object, `g_j^0` is one Gaussian splat, and each Gaussian carries a canonical per-Gaussian state.
- Synchronize Module 1 notation by writing `G^0=\{g_j^0\}_{j=1}^{N_G}`, using `\alpha_j^0` in opacity reliability, and outputting `\widetilde{G}^0=\{(g_j^0,q_j^\alpha,n_j^G)\}_{j=1}^{N_G}`.
- Expand Module 1 `Operation`, `Output`, and `Effect` to explain covariance as an anisotropic shape descriptor, `n_j^G` as an uncertain Gaussian normal candidate, `q_j^\alpha` as a continuous opacity reliability weight, and `\widetilde{G}^0` as an enriched Gaussian set rather than a new rendering asset.
- Expand the remaining pipeline modules in the same style:
  Module 2-7 now explain proxy extraction, simulation-oriented remeshing, boundary/geometry/Gaussian demand fields, anchors/load samples, persistent binding, Blender supervision, and conservation-preserving projection.
- Expand runtime Module 8-15 to explain wind queries, exposure, force-space load prediction, load-to-anchor projection, XPBD solving, Gaussian attribute transport, SH/appearance correction, and final 3DGS rendering.
- Add clarifying text to the runtime loop, minimum development version, module hierarchy, and design dependency sections so the document distinguishes precomputed persistent state from per-frame runtime values.
- Revise opacity reliability notation from fixed `P20/P80` thresholds to generic `P_low/P_high` percentile thresholds, with `P20/P80` described only as an MVP heuristic default and ablation setting.
- Add a Module 1 reference/originality table that marks standard 3DGS representation and covariance construction as reference-based, normal-candidate use as reference-based adaptation, opacity reliability as heuristic/original supporting design, and the enriched Gaussian set as pipeline-specific design.
- Add citation tags for 3DGS, SuGaR, and GOF in the pipeline note and append a local bibliography using `ideas/refs.bib`.
- Fix the reference/originality table layout by wrapping multi-label cells in local stacked macros, preventing `[Ours]` from being parsed as a new table row.
- Replace the centralized `Reference and originality note` table with inline reference/originality paragraphs placed directly after the relevant Module 1 equations, so the reader does not need to jump between a summary table and the equations.
- Add an explicit `kerbl2023gaussian` citation to the covariance-construction reference paragraph.
- Switch the Latin fonts from Liberation to DejaVu in this note so the local XeLaTeX build succeeds in the current environment.
- Revise `Module 2: Proxy Mesh Extraction` so GOF-style extraction is the main raw proxy extraction path for real trained 3DGS assets, while SuGaR-style and TSDF-style extraction are comparison baselines.
- Add a Module 2 development plan that treats mesh extraction as an independently testable module with a shared adapter interface, GOF-first implementation, raw mesh quality reporting, and later SuGaR/TSDF comparison adapters.
- Add a TSDF/volumetric fusion reference entry for Curless and Levoy 1996 in `ideas/refs.bib`.

## Changed Files

- `ideas/3dgs_wind_response_pipeline_2026-05-12.tex`
- `ideas/3dgs_wind_response_pipeline_2026-05-12.pdf`
- `ideas/refs.bib`

## Verification

- Ran `xelatex -interaction=nonstopmode -halt-on-error 3dgs_wind_response_pipeline_2026-05-12.tex`.
- Re-ran `xelatex -interaction=nonstopmode -halt-on-error 3dgs_wind_response_pipeline_2026-05-12.tex` after expanding the remaining modules.
- Ran `bibtex 3dgs_wind_response_pipeline_2026-05-12` after adding citations, then re-ran XeLaTeX twice.
- Result: PDF build succeeded and wrote `ideas/3dgs_wind_response_pipeline_2026-05-12.pdf`.
- Ran `xelatex`, `bibtex`, and two further `xelatex` passes after adding the GOF/SuGaR/TSDF Module 2 recommendation and the Curless-Levoy TSDF citation.
- Resulting pipeline note is 29 pages after replacing the reference/originality table with inline notes and adding the Module 2 extraction recommendation.
- Remaining LaTeX output only reports an underfull hbox warning on the long Module 7 title; no build failure, undefined citations, or overfull boxes.

## Next

- Continue using this document as the detailed pipeline reference when implementing the next milestone.
- Start implementation from the reusable mesh extraction module under `code/wind3dgs/`, with thin experiment wrappers only after the shared interface and GOF adapter are defined.
- If the pipeline note becomes paper-facing, do one visual pass for page breaks and table density.

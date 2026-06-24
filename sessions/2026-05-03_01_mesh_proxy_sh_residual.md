# 2026-05-03 mesh proxy and SH residual direction

## Context

Discussed whether the project should use direct 3DGS input, a mesh proxy, or a hybrid representation for wind-driven deformable bodies. Also discussed whether predicting SH coefficients with AI is a convincing contribution.

## Decisions

- Keep the final input as a trained 3DGS asset.
- Internally extract or construct a lightweight simulation mesh proxy for topology-aware wind deformation.
- Bind Gaussians to the mesh proxy using local frames, barycentric coordinates, and offsets.
- Avoid positioning the method as direct Gaussian-patch aerodynamics because that overlaps with Gaussian Swaying.
- Use AI for deformation-aware SH residual prediction, not full SH coefficient prediction from scratch.
- Frame the SH module as a fast appearance correction method that avoids expensive per-frame SH re-optimization.
- Updated the shared research workflow so future edits to `ideas/idea_sketch.tex` must be followed by a PDF build attempt.
- Reviewed `paper/wind_deformable_3dgs_algorithm_v8.pdf` and incorporated applicable equations into the current mesh-proxy formulation.
- Rewrote ED-Graph/DQS-specific formulas as mesh-proxy vertex dynamics, Gaussian-to-triangle binding, and local-frame transport.
- Clarified that the technical structure can stay as-is, but the paper should be framed around topology-aware wind-deformable 3DGS representation rather than AI-based SH coefficient prediction.
- Added the argument that the method should be defended as a necessary representation bridge, not a loose combination of known components.
- Identified adaptation novelty points: Gaussian attribute transport, simulation-oriented proxy extraction, Gaussian-opacity wind exposure, dense-to-proxy force projection, and deformation-aware SH residual transport.
- Added a shared workflow principle that the AI agent should act as an objective, strategic research partner for paper acceptance, including pushing back on weak ideas instead of merely matching the user's intent.

## Changed Files

- `ideas/idea_sketch.tex`
- `ideas/refs.bib`
- `ideas/changelog.md`
- `paper/wind_deformable_3dgs_algorithm_v8.pdf`
- `../RESEARCH_PROJECT_GUIDE.md`
- `../templates/research_project/TEMPLATE_MANIFEST.md`
- `../templates/research_project/AGENTS.template.md`
- `AGENTS.md`
- `sessions/2026-05-03_mesh_proxy_sh_residual.md`

## Next

- Prototype a synthetic mesh-to-Gaussian binding example.
- Test mesh deformation under procedural wind.
- Compare canonical SH, analytic local-frame SH correction, learned residual SH correction, and per-frame SH optimization.

## Build Notes

- Built `ideas/idea_sketch.pdf` successfully after LaTeX tools were installed.
- Build command: `latexmk -pdf -interaction=nonstopmode -halt-on-error -outdir=ideas ideas/idea_sketch.tex`
- Output: `ideas/idea_sketch.pdf`
- Rebuilt `ideas/idea_sketch.pdf` successfully after importing the v8 equations.
- Rebuild again after the contribution hierarchy and framing update.
- Rebuild again after the necessity and adaptation novelty update.
- Rebuilt `ideas/idea_sketch.pdf` successfully after the necessity and adaptation novelty update.

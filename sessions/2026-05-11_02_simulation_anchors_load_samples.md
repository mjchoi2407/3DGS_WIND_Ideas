# 2026-05-11 simulation anchors and load samples

## Context

The user provided `ideas/anchor_method_revision_notes_2026-05-11_v3.tex` as a detailed reference and asked to compress its content into `ideas/idea_sketch.tex` without losing the reference trail.

## Decisions

- Keep `ideas/anchor_method_revision_notes_2026-05-11_v3.tex` as the detailed design reference.
- Add the compressed paper-level framing to the idea sketch.
- Replace raw Gaussian-density-based anchor placement with reliability-weighted Gaussian transport demand.
- Treat isolation only through a view-masked gated floater likelihood, not as an independent reliability term.
- Separate persistent simulation anchors from fixed material-space load samples.
- Use dense-force-supervised projection risk for one-time fixed load-sample refinement when Blender or simulator dense forces are available.

## Changed Files

- `ideas/idea_sketch.tex`
- `ideas/changelog.md`
- `ideas/idea_sketch.pdf`

## Verification

- Ran `xelatex -interaction=nonstopmode -halt-on-error idea_sketch.tex`.
- Re-ran `xelatex -interaction=nonstopmode -halt-on-error idea_sketch.tex`.
- Re-ran once more after the wording cleanup.
- Result: PDF build succeeded and wrote `ideas/idea_sketch.pdf`.

## Next

- Implement the MVP anchor/load sampler under `code/wind3dgs/`.
- Validate ablations for raw Gaussian density, direct isolation reliability, fixed load samples, and projection-risk-aware load-sample refinement.

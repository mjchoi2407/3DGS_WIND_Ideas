# 2026-05-18 scope checklist update

## Context

The user clarified that the system may have too many components, but SH residual correction should not be removed because it enables runtime rendering without per-frame SH optimization. The implementation checklist needed to reflect this reduced but SH-preserving scope.

## Decisions

- Keep SH residual correction as a required runtime appearance module.
- Keep the paper framing centered on runtime wind-deformable 3DGS representation, not AI SH prediction.
- Treat advanced anchor/load scoring as a robustness/refinement path rather than the MVP critical path.
- Start M4/M7/M8 with fixed simulation anchors and simple fixed material-space load samples.
- Preserve projection-risk-aware load-sample refinement as optional supervised refinement after the first solver/rendering loop works.
- Follow-up audit confirmed that the v3 anchor notes are reflected in the idea sketch, and the idea sketch is reflected in the checklist.
- Tightened two weak spots: idea sketch now says SH residual is runtime-required but not the headline contribution, and M6 now includes reliability-weighted Gaussian transport demand plus optional view-masked gated floater refinement.
- Cleaned up load-sample notation from the ambiguous `q_l=(T_l,b_l)` form to \(q_\ell := (\tau_\ell,\mathbf{b}_\ell)\), where \(\tau_\ell\) is a proxy face index and \(\mathbf{b}_\ell\) is the barycentric coordinate.
- Audited formula notation across v3, the idea sketch, and the checklist. Replaced the overloaded Gaussian binding notation `T_j` / `R_{T_j}` / `\Delta T_j` with \(\tau_j\), \(\mathcal{F}_j\), and \(\Delta R_j\). Synced load-sample effective wind to use current velocity \(v_\ell(t)\), and renamed `theta_scene` in the load model to `theta_field`.

## Changed Files

- `ideas/implementation_checklist.md`
- `ideas/implementation_checklist.tex`
- `ideas/implementation_checklist.pdf`
- `ideas/idea_sketch.tex`
- `ideas/idea_sketch.pdf`
- `ideas/anchor_method_revision_notes_2026-05-11_v3.tex`
- `ideas/changelog.md`

## Verification

- Ran `xelatex -interaction=nonstopmode -halt-on-error implementation_checklist.tex`.
- Re-ran after wording cleanup.
- Result: PDF build succeeded and wrote `ideas/implementation_checklist.pdf`.
- Ran `xelatex -interaction=nonstopmode -halt-on-error idea_sketch.tex`.
- Result: PDF build succeeded and wrote `ideas/idea_sketch.pdf`.

## Next

- Begin M4 with a simple solver, fixed proxy-vertex simulation anchors, and simple fixed load samples.
- Keep M9 as a required runtime milestone before final paper evidence.

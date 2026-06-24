# 2026-05-11 controllable fields

## Context

The discussion separated two ideas that had both been called "anchors": physical attachment constraints and reduced simulation control nodes. The user asked to first record the controllable physical/material field direction in the idea sketch, then return to reduced simulation control anchors.

## Decisions

- Reframe constraint and material setup as a profile-conditioned, semi-automatic, user-editable field initialization layer.
- Treat attachment anchors as physical boundary or handle constraints.
- Treat simulation control nodes as reduced degrees of freedom for efficient simulation; this remains the next design topic.
- Add field categories: attachment/support, material compliance, feature/structure, wind response/exposure, and external force/handle.
- Keep automation as field initialization from object profile, geometry cues, Gaussian support, and user edits rather than a universal automatic anchor detector.

## Changed Files

- `ideas/idea_sketch.tex`
- `ideas/changelog.md`
- `ideas/idea_sketch.pdf`

## Verification

- Ran `xelatex -interaction=nonstopmode -halt-on-error idea_sketch.tex`.
- Ran `bibtex idea_sketch`.
- Re-ran `xelatex -interaction=nonstopmode -halt-on-error idea_sketch.tex`.
- Result: PDF build succeeded. LaTeX reported one small overfull hbox warning in the formula transfer notes, but no build failure.

## Next

- Define sampling strategies for reduced simulation control nodes separately from attachment/material field initialization.

# Wind3DGS Ideas Instructions

This is the ideas-focused repository inside the Wind3DGS project workspace.

Project-specific topic: CG + AI research on wind-driven deformable 3D Gaussian Splatting.

Project tag for conversation/session tracking: `Wind3DGS`.

Sibling work folders:

- `../code`: reusable implementation, configs, scripts, dependencies, and code-side session notes
- `../ideas`: idea sketches, checklists, bibliography, research direction changelog, and idea-side session notes
- `../experiments`: experiment READMEs, assets, outputs, reports, wrappers, and experiment-side session notes

## Startup Protocol

At the start of every meaningful task:

1. Read `../RESEARCH_PROJECT_GUIDE.md` if available from the workspace root.
2. Read this `AGENTS.md`.
3. Check local idea-side records:
   - `idea_sketch.tex`
   - `changelog.md`
   - `refs.bib`
   - `sessions/README.md`

## Working Loop

1. Keep early research decisions in `idea_sketch.tex`.
2. Record research pivots in `changelog.md`.
3. Keep implementation changes in `../code/`.
4. Keep experiment records and generated outputs in `../experiments/`.
5. Record idea-side work history in `sessions/`.

## Editing Rules

- Preserve existing user files unless explicitly asked to reorganize them.
- When a paper idea changes, update `idea_sketch.tex` and add a dated entry to `changelog.md`.
- After editing `idea_sketch.tex`, build `idea_sketch.pdf` to verify the LaTeX output. If the build fails or LaTeX tooling is unavailable, report the reason and record it in the session note.
- Keep implementation details out of this repo unless they are pseudocode, design notes, or experiment requirements.

## Session Tracking

- Start substantial new conversations with a prefix like `[Wind3DGS | idea]` or `[Wind3DGS | refs]`.
- At the end of meaningful idea work, create or update a note under `sessions/`.
- Name new session notes as `YYYY-MM-DD_NN_short_topic.md`, where `NN` is the next two-digit sequence for that date inside `ideas/sessions/`.
- Keep numbering independent from `../code/sessions/` and `../experiments/sessions/`.
- Do not rename legacy unnumbered notes unless the user explicitly asks for a migration.
- Use `changelog.md` for research direction changes, not ordinary conversation history.

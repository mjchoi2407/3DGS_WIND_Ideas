# 2026-06-24 project internal split cleanup

## Context

The split repository setup was corrected so the three Git repositories live inside the original project directory instead of as sibling folders outside it.

## Decisions

- Use `ideas/` as the independent ideas Git repository inside `Wind_Deformable_3DGS/`.
- Keep idea-side session history in `ideas/sessions/`.
- Configure `ideas/` with `origin` set to `git@github.com:mjchoi2407/3DGS_WIND_Ideas.git`.
- Leave commit and push for later review.

## Changed Files

- `AGENTS.md`
- `.gitignore`
- `README.md`
- `sessions/`

## Verification

- `git -C ideas remote -v` shows the expected SSH remote.
- `find ideas -maxdepth 2 -type d -name sessions` shows `ideas/sessions`.

## Next

- Review, then run `git -C ideas add -A && git -C ideas commit -m "Initial ideas repository"` when ready.

# 2026-06-25 01 initial split repo push

## Context

The user asked to review today's session history, summarize the work, and push the split Git repositories.

## Reviewed Sessions

- `sessions/2026-06-24_01_project_internal_split_cleanup.md`
- `sessions/2026-06-24_02_session_numbering_rules.md`

## Summary

- The Wind3DGS workspace now keeps research framing and planning in the independent `ideas/` repository.
- The ideas repository remote is `git@github.com:mjchoi2407/3DGS_WIND_Ideas.git`.
- Idea-side session notes were moved under `ideas/sessions/` and renamed into the numbered `YYYY-MM-DD_NN_short_topic.md` convention.
- The repository contains the living idea sketch, implementation checklist, bibliography, changelog, and generated PDFs for the current research direction.
- Ordinary conversation/task history belongs in `ideas/sessions/`; research-direction pivots belong in `changelog.md`.

## Push Scope

- Initial ideas repository files.
- Research notes, TeX/PDF documents, checklist, bibliography, changelog, and idea-side session records.

## Next

- Commit this repository and push `main` to `origin`.
- Keep future idea-side work history in `ideas/sessions/`.

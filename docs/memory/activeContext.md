> Last updated: 2026-03-25 (session 3)

## Current focus

Workshop PR deliverables: `docs/product/PRD.md` is done (v0.1, In Review). Only `.cursorignore` remains before the PR can be opened.

## What was just done

- Delivered Audit 3 report (0 Critical, 2 Major, 3 Minor, 2 Suggestions).
- Applied all Audit 3 fixes across 4 files:
  - `CLAUDE.md` — added `packages/utils/` to monorepo table; moved `decisionLog.md` from pre-loaded to reference docs
  - `docs/memory/progress.md` — corrected domain-glossary term list to actual 20 entries
  - `docs/memory/decisionLog.md` — restored chronological order (two ~2020 entries moved before ~2022 Jotai entry); updated entry 7 to document the size-cap exemption for decisionLog itself
  - `docs/memory/activeContext.md` — refreshed stale yarn.lock expiry note
- Created `docs/product/PRD.md` — reverse-engineered PRD of Excalidraw v0.1 (In Review), covering problem statement, 5 goals, 12 requirements (P0/P1/P2), metrics with baselines and targets, user journeys, risks, 3-phase launch plan, and 4 open questions with owners/deadlines.
- Updated `progress.md` and `activeContext.md` to mark PRD as done.

## Active decisions

- `docs/memory/` files are capped at ~200 lines — they are loaded every session; keep them scannable, not exhaustive.
- `docs/memory/decisionLog.md` is exempt from the pre-loaded cap — it is append-only and lives in the reference docs list in `CLAUDE.md`.
- `docs/technical/` and `docs/product/` files are reference docs — longer is fine, depth over brevity.
- All docs are facts-only, source-verified — no assumptions or inferred behavior without a cited file/line.
- Memory Bank files cross-reference each other at the top rather than duplicating content.

## In progress (not finished)

- **`.cursorignore`** — not yet created; last remaining required PR checklist item.
- **Implementation work** — 0% — no feature tasks started; awaiting user direction.

## Known issues & open questions

*Issues*:
- `docs/memory/techContext.md` and `systemPatterns.md` were modified by the user after creation (cross-reference links added at top) — treat user edits as authoritative, do not overwrite.
- `frame.test.tsx` FIXME: frame detection passes in browser, fails in jsdom — root cause unknown, do not attempt to fix without more context.
- Arrow label version-bump test (`textWysiwyg.test.tsx:335`) is non-deterministically flaky — do not mark as fixed without a reproducible root cause.

*Questions* (also tracked in `docs/product/PRD.md` Open Questions):
- Q-1: What is the minimum supported browser version? Check `excalidraw-app/vite.config.ts` `build.target` — Owner: Engineering Lead.
- Q-2: What is the maximum acceptable gzipped bundle size for `@excalidraw/excalidraw`? — Owner: Engineering Lead.
- Q-3: Does the collaboration server encrypt scene data in transit and at rest? — Owner: Infrastructure.
- Q-4: Is `docs/product/PRD.md` a reverse-engineered PRD of existing Excalidraw, or a PRD for new work built on top? — Owner: Workshop Facilitator/PM.

## Next steps

1. Create `.cursorignore` in repo root — last required PR deliverable.
2. Open the Workshop PR with the PR template filled.
3. Identify the first implementation task — needs user direction; no code has been changed yet.
4. Run `yarn test:all` to establish a clean baseline before any code changes.

## Context that expires

- `yarn.lock` shows `M yarn.lock` in git status (observed across multiple sessions) — check before running `yarn install` or adding dependencies; root cause not yet investigated.
- No feature flags or temporary workarounds are currently active in the codebase (from this session's perspective).

> Last updated: 2026-03-25 (session 5)
> Related: [progress.md](progress.md) | [decisionLog.md](decisionLog.md) | [PRD](../product/PRD.md) | [architecture](../technical/architecture.md) | [undocumented-behavior](../technical/undocumented-behavior.md)

## Current focus

All Workshop PR deliverables complete. Ready to open PR.

## What was just done

- Created `.cursorignore` (copy of `.claudeignore`) — last required PR checklist item.
- Committed `.claudeignore` to git.
- All required deliverables for the Workshop PR are now done: Memory Bank, PRD, `.cursorignore`, `.claudeignore`.

## Active decisions

- `docs/memory/` files are capped at ~200 lines — they are loaded every session; keep them scannable, not exhaustive.
- `docs/memory/decisionLog.md` is exempt from the pre-loaded cap — it is append-only and lives in the reference docs list in `CLAUDE.md`.
- `docs/technical/` and `docs/product/` files are reference docs — longer is fine, depth over brevity.
- All docs are facts-only, source-verified — no assumptions or inferred behavior without a cited file/line.
- Memory Bank files cross-reference each other via relative markdown links, not plain text.

## In progress (not finished)

- **Workshop PR** — deliverables complete; PR not yet opened.
- **Implementation work** — 0% — no feature tasks started; awaiting user direction.

## Known issues & open questions

*Issues*:
- `docs/memory/techContext.md` and `systemPatterns.md` were modified by the user after creation (cross-reference links added at top) — treat user edits as authoritative, do not overwrite.
- `frame.test.tsx` FIXME: frame detection passes in browser, fails in jsdom — root cause unknown, do not attempt to fix without more context.
- Arrow label version-bump test (`textWysiwyg.test.tsx:335`) is non-deterministically flaky — do not mark as fixed without a reproducible root cause.
- `fixBindingsAfterDuplication()` uses `Object.assign` bypassing `mutateElement()` — latent collaboration bug: binding ref updates on duplicated elements may not propagate to remote clients (version not bumped). See `decisionLog.md` — Object.assign binding bypass entry.

*Questions* (also tracked in [docs/product/PRD.md](../product/PRD.md) Open Questions):
- Q-1: Minimum supported browser version — check `excalidraw-app/vite.config.ts` `build.target` — Owner: Engineering Lead.
- Q-2: Maximum acceptable gzipped bundle size for `@excalidraw/excalidraw` — Owner: Engineering Lead.
- Q-3: Collaboration server encryption — Owner: Infrastructure.
- Q-4: PRD scope — reverse-engineered vs. new work — Owner: Workshop Facilitator/PM.

## Next steps

1. Create `.cursorignore` in repo root — last required PR deliverable.
2. Open the Workshop PR with the PR template filled.
3. Identify the first implementation task — needs user direction; no code has been changed yet.
4. Run `yarn test:all` to establish a clean baseline before any code changes.

## Context that expires

- `yarn.lock` shows `M yarn.lock` in git status (observed across multiple sessions) — check before running `yarn install` or adding dependencies; root cause not yet investigated.
- No feature flags or temporary workarounds are currently active in the codebase (from this session's perspective).

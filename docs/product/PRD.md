| Field        | Value                                                         |
|--------------|---------------------------------------------------------------|
| Status       | In Review                                                     |
| Version      | 0.1                                                           |
| Author       | Workshop Participant (reverse-engineered from codebase)       |
| Reviewers    | Engineering Lead, Design Lead, Workshop Facilitator           |
| Created      | 2026-03-25                                                    |
| Last updated | 2026-03-25                                                    |
| Milestone    | Workshop PR — Memory Bank complete                            |

---

## TL;DR

Excalidraw is a browser-based collaborative whiteboard that renders diagrams in a hand-drawn aesthetic, requiring no account, installation, or design skill to start. It serves anyone who needs to communicate an idea visually in real time — engineers, product managers, and teachers — by removing the friction and false polish of traditional diagramming tools. Success is measured by a user completing a shareable diagram in under 60 seconds from a fresh browser tab and by pointer-event latency staying below 200ms during drawing. Pixel-precise design, rich text, slideshow modes, and live data bindings are out of scope.

---

## Problem statement

### The situation

Digital diagramming tools — Lucidchart, draw.io, Miro, Figma FigJam — require sign-in, impose formal shape libraries and alignment grids, and produce output that looks polished and final. Remote and hybrid teams communicate increasingly through async visual artifacts, but no lightweight, zero-install option existed that preserved the "rough sketch" quality of whiteboard thinking.

### The problem

Engineers, product managers, and teachers routinely need to externalize an idea visually while on a call or composing an async message. With traditional tools, starting a new diagram requires signing in, picking a template, and learning a toolbar — a minimum of 3–5 minutes before any thinking can happen. The resulting artifact looks finished, which signals to collaborators that the idea is settled and discourages iteration. Sharing requires the recipient to either have an account or receive a static image they cannot edit. Teams either default to verbal descriptions (lossy) or accept the overhead of formal tools (slow), both of which impede early-stage design and architectural communication.

### Why now

Browser canvas APIs now support 60fps rasterization of thousands of vector elements. WebSocket infrastructure (Socket.io, Cloudflare) is cheap enough to provide real-time multiplayer at zero marginal cost for small sessions. The RoughJS library made a hand-drawn aesthetic trivially achievable without custom rendering. Remote-first work norms (accelerated post-2020) created a permanent need for shareable visual scratchpads that work in a browser without IT involvement. All three preconditions converged; no equivalent open-source product existed at the time of Excalidraw's initial release.

---

## Goals and non-goals

### Goals

1. Zero-friction start for users: a user with no prior Excalidraw experience must reach a drawing canvas and produce a shareable shape within 60 seconds of opening a fresh browser tab.
2. 60fps canvas rendering for primary users: pointer-event response must remain below 200ms during drag, resize, and freehand drawing at up to 500 simultaneously visible elements.
3. Shareable collaboration for small teams: two collaborators must see each other's cursor positions and element changes within 500ms at typical broadband speeds (10+ Mbps), without either party creating an account.
4. Round-trip export fidelity: a scene exported to PNG must re-import into the editor with full scene data intact, preserving element positions, types, styles, and text.
5. Embeddable library for developers: a third-party developer must be able to mount `@excalidraw/excalidraw` inside their React app with zero global side effects and full isolation between multiple instances on the same page.

### Non-goals

- **Not a vector design tool**: no Bezier path editing, no typography controls, no pixel-level snapping or constraints. Exact measurements are secondary to speed.
- **Not a document editor**: no rich text, pagination, structured layout engine, or versioned document history beyond in-session undo/redo.
- **Not a presentation tool**: no slide metaphor, transitions, speaker notes, or audience mode.
- **Not a data visualization library**: charts, graphs, live data bindings, and formula-driven shapes are out of scope.
- **Not a mobile-first product**: canvas interaction is optimized for pointer and stylus input. Mobile is supported but not the primary design target; mobile-specific gestures and small-screen layouts are secondary P1/P2 concerns.

### Success metrics

**Primary metrics** (ship / no-ship gates)

| Metric | Baseline | Target | Measurement | Timeline |
|--------|----------|--------|-------------|----------|
| Time to first shareable diagram | Not measured (new product) | < 60 seconds from fresh tab | Session replay, funnel analytics | 4 weeks post-launch |
| Pointer-event frame latency (p95) | Not measured | < 200ms during drag/resize | Browser performance API, CI benchmark | Before launch |
| Collaboration cursor delivery latency | Not measured | < 500ms at 10+ Mbps | Socket.io event timestamps, manual QA | Before launch |

**Secondary metrics** (directional signals)

| Metric | Baseline | Target | Measurement |
|--------|----------|--------|-------------|
| Export completion rate | Unknown | > 80% of sessions that reach 3+ elements | Analytics event: export_completed |
| Library install / embed adoption | 0 (new package) | 500 npm downloads/week by week 8 | npm download stats |
| Returning users (sessions > 1) | Unknown | > 30% within 7 days | Session analytics |

**Guardrail metrics** (must not worsen)

| Metric | Threshold |
|--------|-----------|
| Static canvas render time for 500 elements | Must not exceed 16ms per frame (60fps floor) |
| `.excalidraw` file round-trip fidelity | 100% — zero element loss on import of any previously exported file |
| Bundle size of `@excalidraw/excalidraw` | Must not grow by more than 5% per minor release without explicit sign-off |

---

## Users and context

### Primary user

**Alex, Engineering Lead at a mid-size software company.** Alex is highly technical, uses a browser all day, and is in 3–5 meetings daily involving architecture or product decisions. During a video call, Alex needs to sketch a system design to explain a proposal. Alex is not a designer and has no patience for toolbar tutorials. Success feels like: opened the tab, drew the diagram, shared the link, colleague is editing it alongside me — all in under two minutes.

Currently blocked by: all available tools require sign-in, produce polished output that over-signals confidence, or require recipients to create accounts to collaborate.

### Secondary users

**Sam, Integration Developer.** Sam is building an internal knowledge base tool and wants to embed a whiteboard component. Sam needs `@excalidraw/excalidraw` to drop into an existing React app without polluting the global CSS namespace, conflicting with other state management, or requiring a separate backend. Sam interacts with the product entirely through the npm package API and the `ExcalidrawImperativeAPI`.

**Casey, Remote Teacher.** Casey uses the web app to illustrate concepts during live online classes. Casey is less technical than Alex, relies on the toolbar UI, and values the hand-drawn aesthetic because it signals to students that the illustration is informal and evolving. Casey's blocker: tools that look "too designed" discourage students from asking questions about the diagram.

### User journey — current state (without Excalidraw)

1. Alex needs to explain a proposed service boundary to a remote colleague during a call.
2. Alex opens Lucidchart — is prompted to sign in; spends 45 seconds authenticating.
3. Alex picks a blank template — the default grid and shape panel are visible; Alex must dismiss onboarding tooltips.
4. Alex drags shapes from the panel, connects them with arrows; the result looks like a formal architecture diagram, not a sketch.
5. Alex clicks Share — the collaborator needs a Lucidchart account to edit; Alex downgrade to "view only" link.
6. The diagram looks final; the colleague treats it as a decision, not a proposal; the intended exploratory conversation never happens.
7. Total elapsed time: ~8 minutes. Quality of early-stage thinking: degraded.

### User journey — future state (with Excalidraw)

1. Alex opens excalidraw.com in a new tab. Canvas is immediately ready — no sign-in, no picker.
2. Alex draws three rectangles, connects them with arrows, types labels. The hand-drawn style signals "this is rough."
3. Alex clicks the Share icon and copies a Live Collaboration link — one action.
4. Colleague opens the link in their own browser — no account needed. Both cursors are visible on screen.
5. Colleague adds a question mark shape to one of the boxes. Alex sees it within half a second.
6. Alex exports to PNG for the meeting notes. The PNG embeds the scene data; tomorrow, Alex can drag the PNG back into Excalidraw and continue editing.
7. Total elapsed time: < 2 minutes. The diagram looked rough intentionally — the conversation stayed exploratory.

---

## Requirements

### Functional requirements

**F-1**: The system must render all core shape types — rectangle, diamond, ellipse, arrow (straight and elbow), line, freedraw, text, and image — using the RoughJS hand-drawn aesthetic on an infinite canvas. Priority: **P0**. Notes: Shape appearance must be consistent across light and dark themes.

**F-2**: The system must allow a user to begin drawing without creating an account, installing software, or selecting a template. Priority: **P0**. Notes: Local scene state must persist to `localStorage` on every change so closing and reopening the tab restores work.

**F-3**: The system must generate a shareable URL containing encoded scene data such that any recipient can open and edit the scene in a browser without authentication. Priority: **P0**. Notes: URL size limit applies; large scenes must use a separate link-sharing backend rather than embedding all data in the URL.

**F-4**: The system must export the current scene to PNG and SVG formats. PNG export must embed scene data as metadata. Priority: **P0**. Notes: Export of an empty canvas is valid and must not error.

**F-5**: The system must re-import a previously exported `.excalidraw` JSON file or a PNG with embedded scene data with 100% element fidelity — no element loss, no style degradation. Priority: **P0**.

**F-6**: The system must function offline after initial page load, including drawing, undo/redo, export, and local persistence. Collaboration features are explicitly unavailable offline. Priority: **P0**. Notes: Requires service worker and PWA manifest.

**F-7**: The system should support real-time multi-user collaboration via a shared session URL, with remote cursors visible to all participants within 500ms of pointer movement. Priority: **P1**. Notes: Session is ephemeral by default; persistence requires separate backend (out of scope for library consumers).

**F-8**: The system should provide a reusable element library allowing users to save named element groups and insert them into any session. Priority: **P1**.

**F-9**: The system should support undo and redo across all element mutations, with no operation limit in a single session. Priority: **P1**. Notes: Remote collaboration updates must never appear in the local undo stack (see `CaptureUpdateAction.NEVER` in `docs/memory/decisionLog.md`).

**F-10**: The `@excalidraw/excalidraw` npm library must expose an `ExcalidrawImperativeAPI` ref allowing host applications to programmatically read and write scene state, trigger exports, and subscribe to change events. Priority: **P1**.

**F-11**: The system may support Mermaid diagram import, converting Mermaid syntax to Excalidraw elements. Priority: **P2**.

**F-12**: The system may provide a command palette (keyboard-activated) for all toolbar actions. Priority: **P2**.

### Non-functional requirements

- **Performance**: Static canvas must render at or above 60fps (≤16ms per frame) for scenes up to 500 visible elements. Interactive canvas (selection handles, cursors) must respond to pointer input within 200ms p95.
- **Export performance**: PNG and SVG export must complete within 2 seconds for scenes up to 500 elements on a 2020-era mid-range device.
- **Availability**: excalidraw.com targets 99.9% monthly uptime. Collaboration server is separate; its downtime must not prevent local drawing or export.
- **Accessibility**: WCAG 2.1 AA for all toolbar, dialog, and sidebar interactions. Canvas drawing interactions are inherently pointer-dependent and are exempt from AA keyboard navigation requirements, but all non-canvas UI must be keyboard and screen-reader accessible.
- **Browser support**: To be confirmed. Check `excalidraw-app/vite.config.ts` `build.target`. See Q-1.
- **Library isolation**: Multiple `@excalidraw/excalidraw` instances on a single page must have fully isolated state — no shared atoms, no shared singletons, no global CSS pollution.
- **File format stability**: The `.excalidraw` JSON format must remain backwards-compatible indefinitely. New fields must be additive; fields must never be removed or renamed without a migration path.
- **Privacy**: No scene data is sent to any server unless the user explicitly initiates sharing or collaboration. Local-only usage produces zero network traffic.

### Out of scope (explicit)

1. Bezier curve editing and precision path tools — see non-goal: not a vector design tool.
2. Rich text with formatting, inline images, or pagination — see non-goal: not a document editor.
3. Slide-based presentation mode — see non-goal: not a presentation tool.
4. Charts, graphs, or live data-bound shapes — see non-goal: not a data visualization library.
5. Backend infrastructure for persistent collaborative sessions — library consumers must provide their own sync backend.
6. Native mobile app (iOS/Android) — web-only; mobile browser is supported but not optimized.
7. AI-powered shape recognition from freehand drawing — out of scope for this version; Magic Frames AI feature is a separate P2 exploration.

---

## Design and UX requirements

### UX principles

1. **Zero to drawing in one action**: The first interaction after opening the app must place a shape on canvas, not open a modal, a sign-in form, or an onboarding tour. Every click before the first shape is drawn is a failure.
2. **Rough signals safe to change**: The hand-drawn aesthetic is a product decision, not a stylistic preference. It must be visually distinct enough that collaborators understand the diagram is a draft, not a deliverable. Do not add polished rendering modes or "professional" themes.
3. **Sharing is one step**: A user must be able to get a shareable artifact (link or file) in a single action from the canvas. Multi-step export wizards are a design failure for this product.
4. **Collaboration is ambient, not intrusive**: Collaborator cursors and changes must be visible without requiring the user to navigate to a "collaboration mode." The user should not need to change their workflow to go multiplayer.

### Key screens and flows

**Canvas (primary surface)**
Purpose: the infinite drawing surface where all element creation and editing happens.
Critical elements: toolbar (shape picker, selection, eraser), zoom controls, undo/redo, share button, scene title. Must not show sign-in prompts or onboarding overlays on first use.

**Share / Export dialog**
Purpose: generate shareable links and export files.
Critical elements: copy link action, Live Collaboration link, PNG export, SVG export, `.excalidraw` file export. Must be reachable in one click from the canvas.

**Library panel**
Purpose: browse, insert, and save reusable element groups.
Critical elements: search, category browse, one-click insert onto canvas, save selection as library item. Must not require account creation.

**Collaboration session**
Purpose: real-time multiplayer on a shared canvas.
Critical elements: visible remote cursors with name labels, follow-mode toggle, participant count. No separate "session setup" flow — joining via link drops the user directly onto the live canvas.

### Accessibility requirements

WCAG 2.1 AA for all non-canvas UI. Canvas drawing is pointer-dependent and is not subject to keyboard-navigation AA requirements, but the following must hold: all toolbar actions must be reachable via keyboard shortcut; all dialogs must trap focus and support Escape to close; color is never the sole means of conveying information (shape types are distinguishable by shape, not only color).

---

## Technical considerations

### Known constraints

- **Rendering architecture**: The five-canvas layer model (StaticCanvas, InteractiveCanvas, NewElementCanvas, SVG layer, React DOM) is fixed. New visual features must map to one of these layers. See `docs/memory/systemPatterns.md`.
- **State management**: Element state is mutable and lives outside React. New features that mutate elements must use `mutateElement()` or `scene.mutateElement()` — not `setState`. See `docs/memory/systemPatterns.md` — Element Mutation Pattern.
- **Jotai isolation**: All Jotai atoms must go through `editor-jotai` or `app-jotai`, never direct `jotai` imports. ESLint enforces this. Violations break multi-instance isolation.
- **File format**: `.excalidraw` JSON is a public contract. Any schema change requires backwards-compatible migration code and version bumping.
- **Collaboration backend**: The web app's collaboration server is not part of this repository. The library exposes sync hooks; consumers wire their own backend. Socket.io client (`4.7.2`) is the current transport in `excalidraw-app`.
- **PWA**: Service worker registration and caching strategy must be maintained for offline P0 requirement.

### Open technical questions

- **Q-1**: What is the minimum supported browser version? — Owner: Engineering Lead — Needed by: before any browser-compatibility work begins — Resolution path: check `excalidraw-app/vite.config.ts` `build.target` field.
- **Q-2**: What is the maximum acceptable `@excalidraw/excalidraw` gzipped bundle size? — Owner: Engineering Lead — Needed by: before publishing 1.0. — Status: No documented threshold; must be established as a guardrail metric.
- **Q-3**: Does the collaboration server support end-to-end encryption for scene data? — Owner: Infrastructure — Needed by: before any enterprise or privacy-sensitive deployment guidance.

### Dependencies

| Dependency | Status | Risk if late |
|------------|--------|--------------|
| RoughJS (hand-drawn rendering) | Done — vendored | Low |
| Socket.io client (collaboration) | Done — `4.7.2` in package.json | Low |
| fractional-indexing (z-order) | Done — integrated | Low |
| jotai + jotai-scope (state isolation) | Done — integrated | Low |
| Collaboration server (excalidraw-app only) | External — not in this repo | Medium — collaboration P1 blocked if unavailable |

---

## Risks and mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Users default to Figma/Miro for familiarity | High | Medium | The zero-install, no-account experience is the differentiator — measure time-to-first-shape and optimize aggressively; if > 60s, run usability sessions — Owner: PM |
| Performance degrades on complex canvases (500+ elements) | Medium | High | StaticCanvas throttling and ShapeCache are in place; establish perf CI benchmark before launch; if p95 frame time exceeds 16ms at 500 elements, hold ship — Owner: Engineering Lead |
| `.excalidraw` file format breaks on import | Low | High | Backwards-compat tests must cover all prior format versions; any schema change requires a migration test — Owner: Engineering |
| Collaboration server outage breaks P1 | Medium | Medium | Collaboration is P1, not P0; local drawing and export remain functional; add user-visible status indicator for collaboration server health — Owner: Infrastructure |
| Scope creep: "while we're here" feature requests mid-sprint | High | Medium | Non-goals section is explicit and referenced in sprint planning; any addition requires PM sign-off and must be weighed against P0 completion — Owner: PM |
| npm library bundle size grows and discourages adoption | Low | Medium | Establish bundle size guardrail metric (Q-2) before 1.0; add bundle size check to CI — Owner: Engineering Lead |

---

## Launch plan

### Phases

**Phase 0 — Internal (current)**
What: Core drawing, export, local persistence, PWA offline.
Access: Engineering team only.
Gate to Phase 1: All P0 functional requirements pass QA; pointer-event latency < 200ms in CI benchmark; export round-trip test passing.
Rollback: Revert to previous commit; no external users affected.

**Phase 1 — Open beta (web app)**
What: Phase 0 + real-time collaboration, library panel, undo/redo.
Access: Public URL, no account required.
Gate to Phase 2: Time-to-first-shareable-diagram < 60s confirmed via session analytics; no critical export regression in first 72 hours.
Rollback: Feature-flag collaboration off; Phase 0 canvas remains live.

**Phase 2 — npm library GA**
What: `@excalidraw/excalidraw` published to npm with stable `ExcalidrawImperativeAPI`.
Access: All npm users.
Gate: API surface reviewed and locked; bundle size guardrail established (Q-2 resolved); integration docs complete.
Rollback: Yank package version; pin consumers to previous minor.

### Launch checklist

1. All P0 functional requirements verified by QA.
2. Pointer-event latency benchmark passing (< 200ms p95 at 500 elements).
3. Export round-trip test passing (PNG re-import fidelity 100%).
4. PWA offline mode verified in Chrome, Firefox, Safari.
5. Accessibility audit: all toolbar and dialog interactions pass WCAG 2.1 AA.
6. Q-1 (minimum browser version) resolved and documented.
7. Analytics events instrumented: `session_start`, `first_shape_drawn`, `share_link_copied`, `export_completed`.
8. Rollback plan confirmed for each phase.
9. Support team briefed on known issues (see `docs/technical/undocumented-behavior.md`).
10. `docs/product/domain-glossary.md` up to date for support and docs team.

### Communication plan

**Internal**: Engineering and design leads review PRD before Phase 0 gate. Support team briefed on known quirks (flushSync, jsdom test failures) before Phase 1. Leadership informed at Phase 2 milestone.

**External**: No press or partner announcement planned for Phase 0–1 (open-source, organic adoption). npm publish announcement via GitHub release notes and project README at Phase 2.

---

## Open questions

- **Q-1**: What is the minimum supported browser version for excalidraw-app and `@excalidraw/excalidraw`?
  Impact: Browser-compatibility work cannot be scoped; users on older browsers may receive broken experience.
  Owner: Engineering Lead.
  Deadline: Before Phase 1 launch.
  Status: Open — Resolution path: check `excalidraw-app/vite.config.ts` `build.target`.

- **Q-2**: What is the maximum acceptable gzipped bundle size for `@excalidraw/excalidraw`?
  Impact: No bundle size guardrail exists; library may grow without a gate.
  Owner: Engineering Lead.
  Deadline: Before Phase 2 (npm GA).
  Status: Open.

- **Q-3**: Does the hosted collaboration server encrypt scene data in transit and at rest?
  Impact: Enterprise and privacy-sensitive deployments cannot be recommended without this answer.
  Owner: Infrastructure / Security.
  Deadline: Before any enterprise-facing documentation is written.
  Status: Open.

- **Q-4**: Is `docs/product/PRD.md` a reverse-engineered PRD of the existing Excalidraw product, or does it describe new work being built on top of it?
  Impact: Scope and requirements differ significantly; this PRD currently describes the existing product.
  Owner: Workshop Facilitator / PM.
  Deadline: Before Phase 1 scope is finalized.
  Status: Open — this document treats it as a reverse-engineered PRD pending clarification.

---

## Appendix

- `docs/memory/productContext.md` — WHY/WHO/WHAT, UX intent, success signals, constraints
- `docs/memory/projectbrief.md` — package structure, key features, dependency order
- `docs/memory/systemPatterns.md` — state management, rendering pipeline, mutation pattern
- `docs/memory/decisionLog.md` — WHY architectural choices were made (monorepo, class component, mutable elements, Jotai isolation, etc.)
- `docs/technical/architecture.md` — Mermaid dependency graph, data-flow sequence diagrams
- `docs/technical/undocumented-behavior.md` — known HACK/FIXME findings, flushSync sites, initialization order dependencies
- `docs/product/domain-glossary.md` — 20 project-specific terms with precise definitions
- RoughJS: https://roughjs.com — hand-drawn rendering library
- fractional-indexing npm package — z-order collision-free insertion

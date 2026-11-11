# 07 — Decisions

Short decision records. New decisions go at the end; never delete one — mark it
*Superseded by ADR-n* instead.

## ADR-1 — Federation instead of a monolith (accepted, 2026-11-11)

**Context:** five domains, one developer working with AI; mature open-source tools
exist for CAD kernels, solvers, physics, and firmware toolchains.
**Decision:** build the glue (manifest, interface objects, adapters, hub, AI
orchestrator) and only the modules without a suitable open-source equivalent.
**Consequences:** depends on other projects' stability; must integrate through open
formats and MCP/CLI so tools can be replaced.

## ADR-2 — One monorepo for our code (accepted, 2026-11-11)

**Decision:** everything we write lives in `hassassin-mechatronics` (this repo).
Third-party tools stay in their own repositories as pinned dependencies; fork only to
patch, and send patches upstream.
**Why:** one person + AI benefits from atomic cross-module changes, shared types,
and end-to-end tests. Splitting a folder out later is easier than merging repos.

## ADR-3 — SolidCraft as the first mechanical tool (accepted, 2026-11-11)

**Decision:** use SolidCraft (MIT/Apache code) for mechanical CAD through STEP, URDF,
and its MCP/CLI. Do not use its brand or UI assets.
**Risks:** young project; no documented 2D drawings yet. Keep an OpenCascade-based fallback.

## ADR-4 — Audience is everyone; AI-guided mode plus expert mode (accepted, 2026-11-11)

**Decision:** two UX layers on the same data (see [00-vision.md](00-vision.md)).

## ADR-5 — Local-first; online and offline/self-hosted (accepted, 2026-11-11)

**Decision:** a project is a plain folder; every feature works offline except those
that inherently need the internet (prices/stock, cloud AI), which degrade gracefully.

## ADR-6 — English for everything in the repository (accepted, 2026-11-11)

**Decision:** docs, code, comments, commits, specs, and UI text are in English;
Indonesian and other languages come later as UI translations.

## ADR-7 — The PCB module moves in later (accepted, 2026-11-11)

**Decision:** `hassassin-ai-pcb-designer` keeps being developed in its own repo for
now and moves to `modules/pcb/` with its history when the owner asks.

## ADR-8 — Name spelling is "Hassassin" (accepted, 2026-11-11)

**Decision:** the name is spelled **Hassassin** everywhere: product (Hassassin
Mechatronics), repository (`hassassin-mechatronics`), packages (`@hassassin/*`), CLI
(`hassassin`), manifest file (`hassassin.project`), cache folder (`.hassassin/`).
This matches the existing PCB module.

## ADR-9 — License: MIT OR Apache-2.0 (accepted, 2026-11-11)

**Decision:** our code is dual-licensed MIT OR Apache-2.0 (`LICENSE-MIT`,
`LICENSE-APACHE`). Contributions are accepted under the same terms.
**Why:** permissive, so anyone can build on it (including commercially); Apache-2.0
adds an explicit patent grant; same terms as SolidCraft and most of the Rust ecosystem.

## ADR-10 — Two-way ECAD ↔ MCAD integration with owners and change proposals (accepted, 2026-11-11)

**Context:** Fusion 360 grew mechanical-first because of Autodesk's history; we built
electronics first because it is our strongest asset. Real products lead from either side.
**Decision:** build order does not define usage order. Every shared value has one
owner; the other side gets a locked constraint; changes travel as change proposals that
are accepted or rejected, never as silent overwrites. Both flows (shape first, board
first) are part of the two-way integration milestone (M3 when written; M5 after the
reorder in ADR-12). Design: [09-ecad-mcad.md](09-ecad-mcad.md).
**Prior art to study:** IDX/EDMD (ProSTEP iViP) incremental ECAD–MCAD exchange.

## ADR-11 — Our own agentic UI with each tool beside it; SolidCraft embedded unmodified (accepted, 2026-11-11)

**Context:** SolidCraft's UI, look, theme, and brand are under a separate licence; its
engine is MIT/Apache. Options were: open SolidCraft as-is (A), rebuild a full CAD UI on
its engine (B), or our own task-focused UI plus the tool itself (C).
**Decision:** option C. The hub has a dashboard, a project wizard, and an agentic
workspace (chat + task list left, tool right). Mechanical steps show SolidCraft's
**unmodified, version-pinned web build, self-hosted** and driven through its public
command API by a bridge in `modules/mech`, so SolidCraft can be updated without changing
our core. Design: [10-hub-ux.md](10-hub-ux.md).
**Conditions:** written confirmation from the SolidCraft maintainers that embedding is
acceptable under `LICENSE-BRAND`; until then keep the separate-window fallback.

## ADR-12 — Reorder the roadmap around the hub experience (accepted, 2026-11-11)

**Decision:** after M2 (PCB move), build the hub shell (M3) and agentic mechanical design
(M4) before two-way integration (M5), because the owner's workflow starts on the
dashboard and in the agentic workspace. The PCB module's agent (Claude API / Claude Code /
Codex backends, chat UI) becomes the base of `packages/agent` instead of a new build.

## Open questions

1. Desktop shell (Tauri or other) and how solvers are bundled for offline use.
2. Local AI model support for offline mode: which runtime, minimum capability.
3. Where requirements live (manifest vs separate file) — see [03](03-project-manifest.md).
4. Entity id stability across CAD rebuilds.
5. Answer from SolidCraft maintainers about embedding (ADR-11).
6. CLI naming once the PCB moves in: `hassassin pcb <tool>` or keep flat tool names.

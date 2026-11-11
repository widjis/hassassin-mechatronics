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

## Open questions

1. Desktop shell (Tauri or other) and how solvers are bundled for offline use.
2. Local AI model support for offline mode: which runtime, minimum capability.
3. Where requirements live (manifest vs separate file) — see [03](03-project-manifest.md).
4. Entity id stability across CAD rebuilds.

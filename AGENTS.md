# AGENTS.md — Working guide for AI agents and contributors

This is the primary reference for **Hassassin Mechatronics**. Read it fully before
changing anything. Details live in [`docs/`](docs/README.md); if this file and `docs/`
disagree, update both in the same change.

## 1. What we are building

An open-source mechatronics design ecosystem — one big project that brings together:

1. **Mechanical** — parametric CAD, assemblies, drawings
2. **Hardware** — schematic, PCB, 3D PCB, through fabrication
3. **Embedded** — firmware, build, flash, debug
4. **Simulation** — circuits, structural/thermal (CAE), and more
5. **Robotics** — kinematics, dynamics, motion planning, control

All modules share **one project**: requirements, BOM, stages and gates, versions, and
**interface objects** that link data across domains (e.g. PCB mounting holes ↔ enclosure
bosses, MCU pins ↔ firmware GPIOs, motor ↔ robot joint).

Audience: **everyone**, including non-technical people. AI is the front door (guided
mode); full editors are there for experts (expert mode).

Read: [docs/00-vision.md](docs/00-vision.md), [docs/01-architecture.md](docs/01-architecture.md).

## 2. Non-negotiable principles

1. **Federation, not a monolith.** We build the *glue* (hub, project manifest, interface
   objects, adapters, AI orchestrator) and the modules that have no good open-source
   equivalent. Domains that already have good open-source tools are connected, not rewritten.
2. **The project is open data.** The manifest format (`hassassin.project`) and our module
   formats are documented in `docs/` and `spec/`. No data is locked inside one tool.
3. **Local-first.** Everything runs offline on one machine; servers (collaboration,
   cloud solvers) are optional and self-hostable.
4. **One path for changes.** Every document change is a small, invertible operation
   (command layer). UI, collaboration, CLI, MCP, and AI all use the same path.
5. **AI works through the same tools as humans**, shows its work as a task list, and
   stops at stage gates for human approval.
6. **Clean room.** Commercial products (Fusion 360, Flux, etc.) are studied only through
   public documentation and observation. Never copy their code, assets, text, icons,
   names, or look. Never copy GPL code into our code.
7. **Permissive licenses for our code.** Linked dependencies must be permissive
   (MIT, Apache-2.0, BSD, Zlib, per-file MPL-2.0). GPL/LGPL/AGPL tools may only be used
   as **separate processes** or through file formats. See
   [docs/06-licensing.md](docs/06-licensing.md).
8. **Respect other projects' brands.** Example: SolidCraft's code is MIT/Apache, but its
   name, logo, icons, UI kit, and themes are under a separate license — do not use them.
   When we show another tool inside the hub (e.g. SolidCraft's web build), it is the
   **unmodified, version-pinned** build driven only through its public API, so upstream
   updates never require changes in our core ([docs/10-hub-ux.md](docs/10-hub-ux.md)).

## 3. Repository layout (monorepo)

```
hassassin-mechatronics/
├── AGENTS.md            ← this file
├── docs/                vision, architecture, modules, manifest, roadmap, licensing, decisions
├── spec/                open schemas (JSON Schema) — project manifest, interface objects
├── packages/
│   ├── platform/        manifest, interfaces, BOM, gates, sync, tool registry
│   └── agent/           cross-module AI orchestrator (MCP client)
├── modules/
│   ├── pcb/             Hassassin PCB (to be moved from the hassassin-ai-pcb-designer repo)
│   ├── mech/            CAD adapter (SolidCraft via STEP/URDF/MCP)
│   ├── embedded/
│   ├── sim/
│   └── robotics/
└── apps/
    ├── hub/             dashboard, project wizard, agentic workspace; web + desktop
    └── cli/             one CLI + MCP server for all modules
```

Current status: **only `AGENTS.md` and `docs/` exist.** Other folders are created when
their work starts — do not create empty folders or placeholder code.
The PCB module moves into `modules/pcb/` **only when the project owner asks for it**
(it is still being developed in the separate repo `~/Documents/Projects/hassassin-ai-pcb-designer`).

## 4. Technology (defaults; change them through a decision record)

- Strict TypeScript (erasable syntax only, so it runs directly on Node), pnpm workspaces,
  Vite, Vitest — same as the PCB module.
- Rust is allowed for heavy parts (geometry kernels, solvers), compiled to WASM when used
  in the browser, managed with Cargo inside its own folder.
- Desktop: a light shell (e.g. Tauri) around the web hub — not decided yet.
- AI: models are called through APIs with keys from environment variables; **never**
  write a key to a file, log, or commit. External agents (Claude Code, Codex) connect via MCP.

## 5. How to work

1. **Pick work from the roadmap** ([docs/05-roadmap.md](docs/05-roadmap.md)). A finished
   milestone = something people can try, with tests, and with docs updated.
2. **Architecture decisions** (choosing a library, changing a format, changing a
   principle) are recorded in [docs/07-decisions.md](docs/07-decisions.md) before or
   together with the code.
3. **Open format first, code second:** manifest/interface changes update
   `docs/03-project-manifest.md` and `spec/` first.
4. **Tests are required** for core logic (manifest, interfaces, adapters, format
   conversion). Verify with real data when available (e.g. the owner's KiCad projects
   in `~/Documents/Hardware Project/` — read only, never modify).
5. **Report honestly:** say what was tested, what was not, and known limitations.
   Limitations are recorded in the roadmap.
6. **Git:** commit only when asked; commit messages in English, short, explaining *why*.
   Never push, force-push, or rewrite history without permission.

## 6. Language

- **Everything written in the repo is in English:** docs, code, identifiers, comments,
  commit messages, UI text, specs. (UI translations, including Indonesian, come later
  through i18n files.)
- Conversation with the project owner happens in Indonesian; translate decisions into
  English when writing them down.

## 7. Owner context

- Built by **one person working with AI** — prefer small steps that finish, automation,
  and regression tests; avoid scope that cannot be maintained.
- Open source so anyone is free to build on it.
- Must work both **online and offline / self-hosted**.

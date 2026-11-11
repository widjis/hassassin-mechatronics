# 01 — Architecture

## Big picture: federation

```
                         ┌───────────────────────────────────┐
  User ──── Hub UI ──────┤  Hub (web + desktop)              │──── AI orchestrator
  (beginner/expert)      │  project · stages · BOM · versions│      (MCP client)
                         └────────────────┬──────────────────┘
                                          │ hassassin.project manifest
               ┌──────────────┬───────────┼────────────┬───────────────┐
               ▼              ▼           ▼            ▼               ▼
          Mech adapter    PCB module   Embedded     Sim adapter    Robotics adapter
               │          (ours)       adapter          │               │
               ▼              │           ▼             ▼               ▼
          SolidCraft          │       PlatformIO/    ngspice,        MuJoCo,
          (STEP/URDF/MCP)     │       Zephyr,        Elmer/CalculiX  ROS 2, OMPL
                              │       probe-rs       (separate processes)
                        .hpcb / KiCad files
```

We **build**: the hub, the platform (manifest, interfaces, BOM, gates), adapters, the AI
orchestrator, and modules that have no suitable open-source equivalent (a browser-based
PCB designer with AI and a rules engine — already built).

We **reuse**: CAD kernels, solvers, physics simulators, firmware toolchains.

## Components

### 1. Project manifest (`hassassin.project`)

One JSON file at the root of the project folder listing every document per domain, the
requirements, interface objects, the combined BOM, stages and gates, and releases.
Specification: [03-project-manifest.md](03-project-manifest.md). A project is **a plain
folder** (can be put in git, zipped, opened offline).

```
rgb-lamp/
├── hassassin.project        manifest
├── requirements.md         requirements (also kept structured in the manifest)
├── electronics/main.hpcb   PCB + schematic document
├── mechanical/case.sc      SolidCraft document
├── mechanical/case.step    neutral export
├── firmware/               PlatformIO / Zephyr project
└── .hassassin/              cache, simulation results, snapshots (optional to commit)
```

### 2. Interface objects

Explicit links between **entities** in two different documents. Examples:

| Interface | Side A | Side B | Checked |
|---|---|---|---|
| `mount` | PCB mounting hole | enclosure boss/hole | position, diameter |
| `cutout` | USB-C connector on the PCB | opening in the enclosure wall | position, size, side |
| `envelope` | tallest component height | enclosure inner space | clearance |
| `pinmap` | MCU pin + schematic net | firmware GPIO definition | name, pin number, function |
| `actuator` | motor + driver (PCB/BOM) | robot joint | torque, speed, direction |
| `sensor` | sensor on the PCB | frame in the robot model | position, orientation |

Each interface stores the **agreed value** and the version of both sides. Adapters read
the actual values; when they differ, the hub marks the interface as *stale* and shows the
change like an ECO (engineering change order): who changed what, and which side has to
follow. The AI can propose the fix; a person approves it.

### 3. Adapters

A small module per tool, all with the same contract:

```ts
interface Adapter {
  kind: 'pcb' | 'mech' | 'embedded' | 'sim' | 'robotics';
  /** Read a document → linkable entities (holes, connectors, pins, joints…). */
  entities(doc: DocRef): Promise<Entity[]>;
  /** BOM lines from this document. */
  bom(doc: DocRef): Promise<BomLine[]>;
  /** Export to a neutral format (STEP, URDF, netlist, …). */
  export?(doc: DocRef, format: string): Promise<FileRef>;
  /** Apply an approved change on this side, when the tool supports it. */
  apply?(doc: DocRef, change: InterfaceChange): Promise<void>;
  /** MCP tools offered (for the AI orchestrator). */
  mcp?: McpServerSpec;
}
```

Adapters talk to tools through, in order of preference: **the tool's own MCP/CLI** →
**open file formats** → a linked library (only if its license is permissive).

### 4. Platform (`packages/platform`)

Built from patterns already proven in the PCB module:

- **Command layer:** a change = small invertible operations; undo/redo, collaboration,
  and AI all use this path.
- **Sync:** the server orders operations; clients apply optimistically and rebase.
  Self-hostable; without a server everything still works locally.
- **Versions and branches:** automatic and named versions, restore, fork. A project
  release = a snapshot of every document plus the manifest.
- **Comments and review**, **permissions** (edit/comment/view).
- **Stages and gates** across domains (below).
- **Tool registry:** every action has one tool definition (name, input schema,
  description) used by the UI, CLI, MCP, and AI alike.

### 5. Project stages and gates

| Stage | Example gate |
|---|---|
| 1. Requirements | Required fields filled (function, power, size, environment, quantity) |
| 2. System architecture | System blocks and cross-domain interfaces defined |
| 3. Domain design | ERC/DRC clean, CAD model valid, firmware builds |
| 4. Integration | All interface objects green (nothing stale or conflicting) |
| 5. Verification | Simulations meet criteria, no open blocking review comments |
| 6. Production | PCB fab files, STL/STEP/drawings, firmware release, priced BOM |

Gates never hard-lock: users may continue with a recorded reason (*override*).

### 6. AI orchestrator (`packages/agent`)

- An MCP client connected to each module's MCP server (our PCB, SolidCraft, etc.) plus
  the platform tools (manifest, interfaces, BOM, gates).
- Works stage by stage with a **visible task list**; each step is an undoable
  transaction; it stops at gates.
- Models are called through APIs with keys from the environment; local models are
  supported for offline mode (with reduced, documented capability).
- External agents (Claude Code, Codex, etc.) can use the same MCP servers.

### 7. Online and offline

| Mode | How it runs |
|---|---|
| **Fully offline** | Desktop app or local `hub`; project = folder; solvers run as local processes |
| **Self-hosted** | One server (hub + sync + solver job queue) inside your own network |
| **Online** | Same as self-hosted, hosted publicly |

Features that need the internet (part prices and stock, cloud AI models) must fail
politely and tell the user, never break the workflow.

## Code structure

See [AGENTS.md §3](../AGENTS.md#3-repository-layout-monorepo). Dependency rules:

```
apps/*     →  packages/agent  →  packages/platform
apps/*     →  modules/*       →  packages/platform
modules/* never import each other directly — they connect through the manifest and interfaces.
```

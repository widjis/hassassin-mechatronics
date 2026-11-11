# 10 — Hub UX: dashboard, project wizard, agentic workspace

The owner's target experience, recorded 2026-11-11. Option C from the mechanical
discussion (ADR-11): **our own task-focused UI and agent, with each domain tool shown
next to the conversation**.

## 1. Dashboard (front page)

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Hassassin Mechatronics                                   [settings] [me] │
│                                                                          │
│   ┌──────────────────────────────────────────────────────────────────┐   │
│   │  Start your project here   → describe it, or pick a project type │   │
│   └──────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│   Apps                                                                   │
│   [Design Mechanical Parts] [Design Hardware] [Embedded] [Simulation]     │
│   [Robotics]                                                             │
│                                                                          │
│   Recent projects                    Templates                           │
│   • RGB bedside lamp (2 h ago)       • RGB lamp  • IoT sensor  • 2-DOF arm │
└──────────────────────────────────────────────────────────────────────────┘
```

- **Apps** open a module directly (expert mode) — e.g. *Design Hardware* opens the PCB
  module on a new or existing document. Apps that are not built yet are shown as
  "coming soon", never as broken links.
- **Start your project here** opens the project wizard (guided mode).
- **Recent projects** and **templates** come from local project folders (offline) or the
  self-hosted server.

## 2. Project wizard

First question: **what are you making?**

| Choice | Creates | Workspace starts in |
|---|---|---|
| Full mechatronics (mechanical + electronics, later firmware) | manifest with mech + pcb (+ embedded) documents, interfaces ready | Agentic mechanical design |
| Mechanical only | manifest with a mech document | Agentic mechanical design |
| PCB only | manifest with a pcb document | Agentic hardware design (PCB module) |
| From a template | the template's folder | its saved stage |

Then a few plain-language questions (what it does, power, size, quantity) that fill the
requirements (stage 1). Everything can be changed later; the wizard never blocks.

## 3. Agentic workspace

Layout modeled on chat-based coding agents (conversation left, work right), designed by us:

```
┌───────────────────────────────┬──────────────────────────────────────────┐
│ Conversation                  │  Tool panel                              │
│                               │                                          │
│ You: a round bedside lamp,    │   ┌──────────────────────────────────┐   │
│ 90 mm, USB-C, phone control   │   │                                  │   │
│                               │   │   SolidCraft (web build, embedded)│   │
│ Agent: plan                   │   │   – or – Hassassin PCB editor     │   │
│  ☑ requirements               │   │                                  │   │
│  ☑ concept shape              │   └──────────────────────────────────┘   │
│  ◻ electronics space estimate │                                          │
│  ◻ board shape                │  Stage bar: Req ▸ Arch ▸ Design ▸ Integr │
│ ▸ 6 tool steps (folded)       │  Interfaces: 4 ok · 1 stale              │
│                               │                                          │
│ [ message…            ] [Send]│                                          │
└───────────────────────────────┴──────────────────────────────────────────┘
```

- **Left:** chat, agent plan as a task list, foldable tool steps, approvals at gates,
  change proposals. Model choice: Claude API, or the user's own **Claude Code / Codex**
  subscription — the PCB module already does this (see §5).
- **Right:** the tool for the current step:
  - mechanical steps → **SolidCraft web build, embedded unmodified**;
  - hardware steps → **Hassassin PCB editor**;
  - later: firmware editor, simulation results, robot simulation.
- The user can always work directly in the right panel; the agent sees the changes.
- Below the panel: stage bar and interface status, shared by all tools.

## 4. Embedding SolidCraft

Goal: SolidCraft updates must not break or require changes in our core.

| Aspect | Design |
|---|---|
| What is embedded | SolidCraft's **unmodified** web build (its own look and name), pinned to a tested version |
| Where it runs | **Self-hosted on the hub's own origin** (e.g. `/tools/solidcraft/<version>/`) in an `<iframe>` — works offline and lets the hub reach the page's command API |
| How the agent drives it | Through the page's published API `window.solidcraft` (`execute`, `request`, `commands`, `inspect`) via a thin bridge in `modules/mech`; the same command ids as SolidCraft's MCP server |
| Files | The bridge opens/saves `.sc` and STEP in the project folder; interface data lives under our reverse-DNS metadata key |
| Updating SolidCraft | Drop in a new web build under a new version folder, run the contract test (commands and parameters we use), switch the pin. No change to platform code unless a command changed |
| Fallbacks | If embedding is not acceptable to the maintainers: open SolidCraft in its own window/tab with the same bridge, or use only its engine (MIT/Apache) compiled to WASM with our own minimal viewer |

Open points (to settle before building M4):
1. **Brand licence.** Embedding the unmodified SolidCraft app inside our product may count as
   "using Brand Material in another program" (`LICENSE-BRAND` §3a). Ask the maintainers in
   writing before shipping; keep the window/tab fallback ready.
2. **Cross-origin.** The public `app.solidcraft.io` cannot be scripted from our page (different
   origin). Self-hosting solves it; an upstream `postMessage` bridge would also allow the
   public URL — worth proposing upstream.
3. **Build size and pipeline** for the web build (WASM, WebGPU) in our release/offline bundle.

## 5. What already exists in the PCB module

The PCB repository (phase 6) already has the agent experience this workspace needs:
chat-style Agent tab (Markdown replies, foldable tool steps, task card), agent loop over
the shared tool layer, backends **Claude API / Claude Code / Codex** (subscription sign-in
from the browser, tools restricted to the project's MCP server), Ask vs Agent mode, live
edits as a collaborator. M3 lifts this into `packages/agent` and `apps/hub` instead of
building it again.

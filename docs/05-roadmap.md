# 05 — Roadmap

Each milestone ends with something people can try, tests, and updated docs. Order
follows dependencies and the target experience in [10-hub-ux.md](10-hub-ux.md):
dashboard → project wizard → agentic workspace with each tool beside the chat.
Reordered on 2026-11-11 (ADR-12).

| Milestone | Name | Result people can try | Status |
|---|---|---|---|
| **M0** | Foundations | `AGENTS.md`, design docs, decisions, SolidCraft evaluation | ✅ |
| **M1** | Project core | `hassassin.project` JSON Schema, `packages/platform` (read/write manifest, documents, interfaces, status computation), CLI `init`, `status`, `check` | |
| **M2** | PCB joins the monorepo | PCB repo moved to `modules/pcb/` with history; docs translated to English; CLI/MCP merged into one `hassassin` command; PCB adapter (entities, BOM) | waits for owner |
| **M3** | Hub shell | Dashboard with apps, **Start your project here**, project wizard (full / mechanical only / PCB only / template), agentic workspace: chat + task list on the left (agent lifted from the PCB module into `packages/agent`), PCB editor on the right; local server; offline | |
| **M4** | Agentic mechanical design | SolidCraft web build self-hosted and embedded in the workspace (pinned version, contract test); bridge in `modules/mech`; the agent builds and edits parts through it; `.sc`/STEP saved in the project folder | needs brand-licence answer |
| **M5** | Two-way ECAD ↔ MCAD | Ownership + change proposals; **STEP export** of the board; demo A: board → enclosure; demo B: shape first → electronics space estimate → locked board outline, holes, keepouts, pinned LED/USB/button positions in the PCB ([09](09-ecad-mcad.md)) | |
| **M6** | Embedded v0 | Firmware project from schematic (pin map), build via PlatformIO, flash via WebSerial; `pinmap` interface | |
| **M7** | Simulation v0 | SPICE from schematic with plots; requirement checks tied to gate 5 | |
| **M8** | Desktop & self-hosting | Desktop app; one-command self-hosted server; sync of whole projects | |
| **M9** | Robotics v0 | URDF from assembly → MuJoCo sim in hub; torque vs motor check | |
| **M10** | Templates & guided mode | Template gallery; full J1 journey for beginners; Indonesian UI translation | |

## Done criteria per milestone

- **M1:** schema validates the example in [03](03-project-manifest.md); round-trip
  read/write keeps unknown fields; tests for interface status computation
  (ok/stale/conflict/broken).
- **M2:** all existing PCB tests pass inside the monorepo; docs in English; one `hassassin`
  CLI and MCP server; the PCB adapter lists outline, holes, and components (with frames
  and heights) as entities.
- **M3:** the wizard creates a project folder with a valid manifest; the workspace opens
  the PCB editor beside the chat; the agent works with all three backends (Claude API,
  Claude Code, Codex) as it does in the PCB module; works with no internet except the
  model call.
- **M4:** the agent builds the upstream enclosure demo inside the embedded SolidCraft and
  saves it to the project; swapping the SolidCraft version needs no change in
  `packages/*` when the commands we use are unchanged (contract test proves it).
- **M5:** both demos run on a real board from the owner's projects. Moving a hole or
  connector on the owning side opens a change proposal on the other side; accepting it
  updates that side, rejecting it keeps the interface stale with the reason recorded.
  In demo B the PCB shows outline/holes as read-only and DRC flags keepout and
  placement violations. Combined-assembly collision and height checks report results
  to the Integration gate.

## Known risks

| Risk | Mitigation |
|---|---|
| SolidCraft brand licence may not allow embedding its UI in our app | Ask the maintainers before M4 ships; fallbacks: separate window with the same bridge, or engine-only WASM with our own viewer |
| SolidCraft is young and changes fast | Pin versions; contract test per release; integrate through its public command API, MCP, and files only |
| Scope too large for one person | Federation; reuse the PCB module's agent, sync, and tools; every milestone usable alone |
| GPL tools in the toolchain | Separate processes only; documented in [06](06-licensing.md) |
| Entity ids unstable across CAD rebuilds | Anchor interfaces on named features, sketches, and parameters ([research](research/solidcraft.md)) |
| AI quality for beginners | Gates and visible task lists; templates as known-good starting points |
| Low disk space on the development machine | Free ~10 GB before Rust/WASM builds (SolidCraft web build, local CLI) |

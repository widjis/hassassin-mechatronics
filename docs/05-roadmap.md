# 05 — Roadmap

Each milestone ends with something people can try, tests, and updated docs. Order
follows dependencies and value for the user journeys in [04](04-user-journeys.md).
The PCB module keeps its own phase plan (phases 5–7) and continues in parallel.

| Milestone | Name | Result people can try | Status |
|---|---|---|---|
| **M0** | Foundations | `AGENTS.md`, design docs, decisions | ✅ docs written |
| **M1** | Project core | `hassassin.project` JSON Schema, `packages/platform` (read/write manifest, documents, interfaces, status computation), `hassassin` CLI: `init`, `status`, `check` | |
| **M2** | PCB joins the monorepo | PCB repo moved to `modules/pcb/` with history; PCB adapter (entities, BOM); STEP export of a board | waits for owner |
| **M3** | Two-way ECAD ↔ MCAD | Mech adapter for SolidCraft (STEP/MCP); ownership + change proposals; demo A: board → enclosure; demo B: enclosure envelope → locked board outline, holes, keepouts, and pinned connector positions in the PCB ([09](09-ecad-mcad.md)) | |
| **M4** | Hub v0 | Web hub: project home, documents, stages & gates, interface board, combined BOM; local server; offline | |
| **M5** | AI orchestrator v0 | Agent connected to PCB + SolidCraft MCP + platform tools; task list; stops at gates; journey J1 steps 1–5 | |
| **M6** | Embedded v0 | Firmware project from schematic (pin map), build via PlatformIO, flash via WebSerial; `pinmap` interface | |
| **M7** | Simulation v0 | SPICE from schematic with plots; requirement checks tied to gate 5 | |
| **M8** | Desktop & self-hosting | Desktop app; one-command self-hosted server; sync of whole projects | |
| **M9** | Robotics v0 | URDF from assembly → MuJoCo sim in hub; torque vs motor check | |
| **M10** | Templates & guided mode | Template gallery; full J1 journey for beginners; Indonesian UI translation | |

## Done criteria per milestone

- **M1:** schema validates the example in [03](03-project-manifest.md); round-trip
  read/write keeps unknown fields; tests for interface status computation
  (ok/stale/conflict/broken).
- **M2:** all existing PCB tests pass inside the monorepo; STEP export opens in
  SolidCraft and another STEP viewer with correct outline, holes, and thickness.
- **M3:** both demos run on a real board from the owner's projects. Moving a hole or
  connector on the owning side opens a change proposal on the other side; accepting it
  updates that side, rejecting it keeps the interface stale with the reason recorded.
  In demo B the PCB shows outline/holes as read-only and DRC flags keepout and
  placement violations. Combined-assembly collision and height checks report results
  to the Integration gate.
- **M4:** the hub runs with no internet connection; all project data stays in the folder.
- **M5:** every AI step is an undoable transaction and visible in history; the agent
  never passes a gate without approval; works with a mock model in tests.

## Known risks

| Risk | Mitigation |
|---|---|
| SolidCraft is very young and may change or stall | Integrate through STEP/URDF/MCP only; pin versions; keep a fallback (OpenCascade-based) |
| Scope too large for one person | Federation; strict milestone order; every milestone usable alone |
| GPL tools in the toolchain | Separate processes only; documented in [06](06-licensing.md) |
| Entity ids unstable across CAD rebuilds | Design entity naming early (M1/M3) and test with real edits |
| AI quality for beginners | Gates and visible task lists; templates as known-good starting points |

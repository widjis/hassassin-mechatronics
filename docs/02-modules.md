# 02 — Modules

Each module section answers: what it covers, what we build vs reuse, what it exposes to
other modules (entities for interface objects), and its first useful version (MVP).
License details are in [06-licensing.md](06-licensing.md).

---

## Hardware (PCB) — `modules/pcb`

**Status: the most mature module — all planned phases (0–7) done in their first
version** (checked 2026-11-11, last commit `0b4b804`). Currently in the separate local
repository `hassassin-ai-pcb-designer` (not pushed); it moves into `modules/pcb/` when the
owner asks (M2).

Already built:
- browser PCB editor with a WebGL2 renderer and a CSS-like **rules engine** (object rules,
  rulesets with selectors, inheritance, computed vs specified values);
- command layer with undo/redo; manual routing; ratsnest; DRC; copper pours;
- schematic editor, netlist, ERC, schematic → PCB sync;
- fabrication export (Gerber X2, Excellon, BOM, pick-and-place);
- KiCad import (`.kicad_pcb`, hierarchical `.kicad_sch`), verified against real boards;
- project stages and gates; shared tool layer; `hassassin` CLI and MCP server;
- real-time collaboration server: comments, versions, fork, roles, live MCP;
- parts and libraries: parametric footprint generators, multi-unit parts with offers,
  KiCad `.kicad_sym`/`.kicad_mod` import and installed-library search, team library,
  BOM with cost per N boards (prices/stock via CSV or manual, no distributor API yet);
- **in-app AI agent** over the same tools: chat-style Agent tab (task card, foldable tool
  steps), Agent/Ask modes, backends Claude API, **Claude Code** or **Codex** (the user's
  own subscription, signed in from the browser, restricted to the project's MCP server);
- autorouter (A*, 45°, vias) and autoplacer, stackup and impedance calculator, net
  lengths, differential pairs, length tuning, 3D board view (three.js, box bodies).

Known gaps that matter for the ecosystem:
- **no STEP export** and no real 3D component models (boxes from courtyards);
- no entity API for outline/holes/connector frames; no constraints coming from another
  document (locked outline, height keepouts, pinned placements);
- requirements and stage gates live inside the `.hpcb` file; in the ecosystem they move
  up to the project manifest, and PCB gates become domain checks (stage 3);
- docs are in Indonesian (repo rule: English); no license file yet (inherits MIT OR
  Apache-2.0 on the move);
- its `hassassin` CLI/MCP becomes part of the ecosystem-wide `hassassin` command;
- Claude Code (proprietary) and Codex (Apache-2.0) are npm dependencies run as separate
  processes — allowed by the licensing policy, documented as such.

Exposes to other modules:

| Entity | Used by |
|---|---|
| Board outline, thickness, mounting holes | mech (`mount`, enclosure) |
| Connector positions, sizes, facing side | mech (`cutout`) |
| Component heights / 3D bodies | mech (`envelope`), 3D view |
| MCU pin ↔ net ↔ function | embedded (`pinmap`) |
| Netlist + part values | sim (SPICE) |
| Motor drivers, sensors, power budget | robotics, sim |
| Electronic BOM lines | platform BOM |

Needed for the ecosystem: **STEP export** of the board (outline + holes + component
bodies), an entity API on top of the existing tool layer, and constraints from
interfaces owned by the mechanical side (locked outline/holes, keepout rules, pinned
placements) — see [09-ecad-mcad.md](09-ecad-mcad.md).

---

## Mechanical — `modules/mech`

Scope: parametric part modeling (sketch, constraints, extrude/revolve, fillet, shell),
assemblies with joints, 2D drawings, import/export (STEP, STL, 3MF), enclosures for PCBs.

**Reuse first: [SolidCraft](https://github.com/bherbruck/solidcraft)** (checked 2026-11-11):
- Rust; desktop app, browser build, headless CLI, and an **MCP server**;
- code dual-licensed MIT / Apache-2.0; **brand and UI** (name, logo, icons, UI kit,
  themes) under a separate license — not to be reused;
- sketches with constraints, feature timeline, assemblies with joints, motion studies,
  interference checks, mass properties; STEP/IGES/3MF/STL/OBJ/glTF; **URDF export**;
- self-described as "early but usable for simple parts"; very young (late 2026);
  no 2D drawing output mentioned in its docs.

Plan:
1. Connect through **files (STEP, URDF) and MCP**, not internal code, so the tool can be
   replaced. Pin a tested SolidCraft version.
2. Build an **enclosure workflow** in the adapter: from PCB entities, generate an
   enclosure (shell, bosses, cutouts) via SolidCraft's MCP/CLI.
3. Contribute upstream (bugs, features we need) instead of keeping a private fork.
4. Fallback options if SolidCraft stalls: browser CAD projects built on OpenCascade
   (e.g. Chili3D, C33D, replicad) — licenses to be checked before use.

Exposes: part/assembly geometry, mounting features, openings, inner volume, mass
properties, joints (for robotics), mechanical BOM lines (parts, fasteners).

MVP: **PCB → enclosure demo** (see roadmap M2).

---

## Embedded — `modules/embedded`

Scope: firmware project generated from the hardware design, editing, building,
flashing, serial monitor, debugging, and MCU emulation for testing without hardware.

Build:
- **Pin map and peripheral configuration generated from the schematic** (MCU pins, nets,
  buses → a board definition / devicetree overlay / header file). This is the main
  differentiator.
- Browser editor (Monaco), project templates (Arduino-style for beginners,
  Zephyr/PlatformIO for experts), and an AI assistant that writes driver code for the
  parts on the board.

Reuse:
- Build: PlatformIO Core, Zephyr, vendor GCC toolchains — run as separate processes
  (locally or on a self-hosted build server).
- Flash/debug: probe-rs, esptool-js (WebSerial in the browser), OpenOCD as a process.
- Emulation: Renode for MCU/board emulation in CI and "try without hardware".

Exposes: GPIO/peripheral assignments (`pinmap`), firmware version, required pins and
timing constraints, power modes (for sim power budgets).

MVP: a project created from a schematic with an ESP32 or RP2040; the pin header is
generated; it builds and flashes over WebSerial.

---

## Simulation — `modules/sim`

Scope, in order of value vs cost:
1. **Circuit simulation (SPICE)** from the schematic: DC operating point, transient,
   AC — results plotted next to the schematic.
2. **PCB electrical/thermal checks:** trace current capacity and temperature rise, IR
   drop on power nets, simple thermal map from component power.
3. **Structural FEA** for enclosures and brackets: linear static stress and deflection.
4. **Thermal** of the assembly (enclosure + PCB heat sources).
5. Later: EM/signal integrity, CFD.

Reuse (all as **separate processes / job queue**, local or server):
- ngspice (circuits), Gmsh (meshing), Elmer or CalculiX (FEA/thermal), openEMS (EM).

Build: problem setup from project data (no manual re-entry), result viewers in the hub,
pass/fail criteria tied to requirements (gate 5), and AI explanations of results for
beginners.

Exposes: results and pass/fail status per requirement.

MVP: SPICE transient of a circuit taken straight from the schematic, with plotted
results and one requirement check (e.g. "LED current ≤ 20 mA").

---

## Robotics — `modules/robotics`

Scope: robot model from the mechanical assembly, actuators and sensors from hardware,
kinematics, dynamics simulation, motion planning, controller design, and deployment to
firmware or ROS 2.

Reuse:
- Model: URDF/MJCF (SolidCraft already exports URDF).
- Physics simulation: MuJoCo (can also run in the browser via WASM builds).
- Kinematics/dynamics: Pinocchio; planning: OMPL / MoveIt 2; middleware: ROS 2.

Build:
- assembling the robot model from project data: joints from mech, motors/encoders
  from the BOM and PCB, limits from part datasheets;
- motion editor (waypoints, trajectories) and a simulation viewer in the hub;
- export of trajectories/controllers to the embedded module or ROS 2 packages.

Exposes: required torques/speeds (back to motor selection), payload, workspace,
sensor placement.

MVP: a 2–3 DOF arm from a SolidCraft assembly → URDF → MuJoCo simulation in the hub,
with joint torques compared against the chosen motors.

---

## Hub — `apps/hub`

Target experience: [10-hub-ux.md](10-hub-ux.md) — dashboard with apps and
*Start your project here*, project wizard, agentic workspace (chat left, tool right).

Scope: project home (create from template or from a description), stage view with
gates, document list per domain, combined BOM with prices/stock, interface status
board (green/stale/conflict), change requests, versions and releases, comments, the
AI chat/agent panel, settings (models, servers, offline mode).

Runs as a web app served by a local or self-hosted server, and as a desktop app
(shell around the same web app) for full offline use.

## CLI — `apps/cli`

One `hassassin` command that wraps the platform and every module's tools; `hassassin mcp`
serves all tools to external agents. Follows the pattern of the PCB module's CLI
(JSON output, `--fail-on-error` for CI).

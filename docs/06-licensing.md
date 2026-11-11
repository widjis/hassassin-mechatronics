# 06 — Licensing and clean room

## Our code

- License for this repository: **to be decided** — recommended **MIT OR Apache-2.0**
  (dual), matching the Rust ecosystem and SolidCraft. See [07-decisions.md](07-decisions.md).
- Contributions are accepted under the repository license.

## Rules for dependencies

| Kind of use | Allowed licenses |
|---|---|
| Linked / bundled into our code (npm, crates, WASM) | MIT, Apache-2.0, BSD-2/3, ISC, Zlib, MPL-2.0 (file-level) |
| Separate process, called through CLI/MCP/files | Any OSI license, including GPL/LGPL/AGPL |
| Data files (part libraries, 3D models) loaded at runtime from the user's install | Allowed; not redistributed by us unless their license permits |

Before adding a dependency: check its license file in the repository (not only a
directory listing), record it in the table below, and note anything special (brand
rules, attribution, patent clauses).

## Clean room

- Commercial tools (Fusion 360, Altium, Flux, SolidWorks, …) are studied only through
  public docs and normal use. We do not copy their code, UI layouts, icons, wording,
  sample data, or names.
- GPL projects (e.g. KiCad, FreeCAD) may be read for file-format behavior only when the
  format is not publicly documented, and their code is never copied. Prefer public
  format docs and example files.
- Brand assets of open-source projects are respected (e.g. SolidCraft's brand license).

## Third-party tools

"Verified" means the license was read from the project's own repository on that date.
Unverified entries are from memory and **must be checked before adoption**.

| Tool | Use | License | How we use it | Verified |
|---|---|---|---|---|
| SolidCraft | Mechanical CAD | MIT OR Apache-2.0 (code); separate brand license | Process: CLI/MCP, STEP/URDF | 2026-11-11 (README) |
| KiCad | File formats (import) | GPL-3.0 | Files only | — |
| OpenCascade | B-rep kernel (fallback) | LGPL-2.1 with exception | Separate process or decision required | — |
| Chili3D, C33D, replicad | Fallback browser CAD | to check | — | — |
| Zephyr | Firmware RTOS | Apache-2.0 | User project dependency | — |
| PlatformIO Core | Firmware build | Apache-2.0 | Process | — |
| probe-rs | Flash/debug | MIT OR Apache-2.0 | Process or library | — |
| esptool-js | Flash ESP32 from browser | Apache-2.0 | Library | — |
| Renode | MCU emulation | MIT | Process | — |
| Monaco Editor | Code editor | MIT | Library | — |
| ngspice | Circuit simulation | BSD-style (mixed) | Process | — |
| Gmsh | Meshing | GPL-2.0+ (with exceptions) | Process | — |
| Elmer FEM | FEA / multiphysics | GPL / LGPL | Process | — |
| CalculiX | FEA | GPL-2.0 | Process | — |
| openEMS | EM simulation | GPL-3.0 | Process | — |
| MuJoCo | Physics simulation | Apache-2.0 | Library or process | — |
| ROS 2 | Robotics middleware | Apache-2.0 | External system | — |
| MoveIt 2, OMPL | Motion planning | BSD-3 | Process / library | — |
| Pinocchio | Kinematics/dynamics | BSD-2 | Process / library | — |
| Tauri | Desktop shell | MIT OR Apache-2.0 | Library | — |
| Three.js | 3D viewer | MIT | Library | — |
| js-angusj-clipper | Polygon clipping (PCB) | MIT | Library (already used) | in PCB repo |

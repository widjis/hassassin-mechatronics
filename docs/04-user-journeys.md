# 04 — User journeys

These journeys drive priorities: a feature matters if it unblocks one of them.

## J1 — Beginner: "RGB bedside lamp" (guided mode)

1. **Start.** Opens the hub, clicks *New project*, types: "An RGB bedside lamp I can
   control from my phone, powered by USB-C."
2. **Requirements.** The AI asks a few plain-language questions (size? battery or USB
   only? how bright?) and fills the requirements. Gate 1 turns green.
3. **System architecture.** The AI proposes blocks: USB-C power, ESP32-C3 module, three
   LED channels with MOSFET drivers, a round enclosure. The user sees a simple block
   diagram and approves it. Gate 2 green.
4. **Electronics.** The AI places parts from the library (with stock and price), draws
   the schematic, runs ERC, creates the PCB, places and routes it, runs DRC. Each step is
   on the task list; the user can open any step, ask "why?", or undo it.
5. **Enclosure.** From the board's mounting holes and USB connector, the AI creates an
   enclosure in the mechanical module, with interface objects for the holes and the USB
   opening. The user changes the diameter with a slider.
6. **Firmware.** A firmware project is generated with the pin map from the schematic and
   a phone-control example. It builds; the user can test it in emulation, then flash it
   over USB from the browser.
7. **Check.** A circuit simulation confirms LED currents; interfaces are all green.
8. **Produce.** One click exports: PCB fabrication ZIP, enclosure STL for 3D printing,
   firmware binary, and a BOM with shop links.

What it needs: hub, templates, PCB module (phases 5–6), mech adapter + enclosure
workflow, embedded pin-map generation, SPICE, AI orchestrator.

## J2 — Engineer: change propagation

1. Opens an existing project (offline, on a laptop).
2. Moves the USB connector 2 mm on the PCB to fix a routing problem.
3. The hub marks `IF-usb` as **stale**: "J1 moved 2.0 mm in +x — enclosure opening
   no longer aligned."
4. The engineer clicks *Propose fix*; the AI updates the opening in the CAD model via
   the mechanical tool; the engineer reviews the diff and accepts.
5. Saves a release "Proto B"; syncs to the team's self-hosted server when back online.

## J2b — Designer: shape first

1. An industrial designer models a handheld remote in the mechanical module and draws
   the PCB envelope inside it: outline, four screw holes, a low-height zone under the
   battery, and required positions for three buttons and an LED.
2. The hub creates `outline`, `mount`, `keepout`, and `placement` interfaces owned by
   the mechanical side.
3. The electronics engineer opens the PCB: the outline and holes are locked, the
   buttons and LED are pinned in place, and DRC flags any part that is too tall in the
   battery zone.
4. The engineer needs 1 mm more width for routing and sends a change proposal on the
   outline; the designer accepts it, and the enclosure updates.

## J3 — Educator: offline lab

1. Installs the desktop app on lab computers without internet access.
2. Distributes a template project folder (zip).
3. Students work through the guided stages with a local AI model (reduced capability)
   or without AI, using templates and checklists.

## J4 — Robotics: small arm

1. Designs a 3-DOF arm in the mechanical module (assembly with joints).
2. Exports URDF; the robotics module builds the model with motors from the BOM.
3. Simulates a pick-and-place motion in MuJoCo; the hub shows that joint 2 needs more
   torque than the chosen motor provides.
4. The AI suggests a gear ratio or a bigger motor; the motor driver board in the PCB
   module and the firmware's control loop are updated through interface objects.

## J5 — External agent

A user with Claude Code or Codex connects to `hassassin mcp` and asks: "add a
temperature sensor and show its value on the phone app". The agent edits the
schematic, PCB, and firmware through the same tools, visible to collaborators as an
*agent* user.

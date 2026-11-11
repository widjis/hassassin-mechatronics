# 04 — User journeys

These journeys drive priorities: a feature matters if it unblocks one of them.

## J1 — Mechatronics: "RGB bedside lamp" (guided, shape first)

The owner's own way of working: visualise in 3D first, then fit the electronics into
the space that is left.

1. **Dashboard → Start your project here.** The wizard asks *what are you making?* →
   **Full mechatronics**. A few questions fill the requirements: USB-C powered, phone
   control over BLE, about 90 mm across, 5 prototypes. Gate 1 green.
2. **Agentic mechanical design.** Workspace opens: chat on the left, SolidCraft on the
   right. "A round lamp with a frosted dome on a wooden-look base." The agent sketches
   and builds the concept; the user adjusts it directly or by asking.
3. **Electronics space estimate.** Before the shape is final, the agent lists the rough
   electrical blocks (USB-C, regulator, BLE module, LED drivers, LEDs) and estimates
   board area, tallest parts, heat, and an antenna keep-clear zone. They appear as
   space boxes inside the 3D model, so the user sees whether the shape works.
4. **Detailed design.** The user refines the base and dome around the space boxes and
   decides on one board in the base (or two: an LED board under the dome and a control
   board).
5. **Generate the board shape.** The PCB envelope comes from the model: outline,
   screw holes, height-limited zones under the dome, antenna zone, and required
   positions for USB-C, the button, and the **LEDs** (an optical decision, owned by the
   mechanical side). Interfaces `outline`, `mount`, `keepout`, `placement`, `cutout`
   are created.
6. **Hardware design inside the limits.** The workspace switches to the PCB editor. The
   agent picks parts (with stock and price), draws the schematic, runs ERC, places and
   routes the board within the locked outline; DRC flags any space violation.
7. **Change proposal if space runs out.** "Need +1.5 mm width for routing" goes back to
   the mechanical side; the user accepts or finds another way.
8. **Firmware.** Pin map from the schematic, colour effects, BLE control; build, test in
   emulation, flash over USB from the browser.
9. **Check.** Collision and clearance in the combined assembly, LED current simulation,
   simple thermal check; all interfaces green.
10. **Produce.** PCB fabrication ZIP, STL for the base and dome, firmware binary, BOM
    with prices.

What it needs: hub shell and wizard (M3), embedded SolidCraft (M4), two-way integration
with space estimate (M5), embedded (M6), simulation (M7).

## J1b — Beginner: board first

Same lamp for someone who starts from the electronics (e.g. they already have a
module): **PCB only** in the wizard, then *add an enclosure* later — the enclosure
follows the board (electronics-led flow in [09](09-ecad-mcad.md)).

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

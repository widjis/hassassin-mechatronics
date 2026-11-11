# 09 — Mechanical ↔ electronics integration (ECAD–MCAD)

How the mechanical module (MCAD) and the hardware module (ECAD) work on the same
product in **both directions**. The same pattern later applies to embedded
(pin maps) and robotics (actuators).

## Build order is not usage order

We build the PCB module first because it is our strongest asset and has no comparable
open-source browser equivalent; mechanical CAD is reused (SolidCraft). Users, however,
start from requirements and system architecture, and the product decides which domain
leads:

| Product type | Usually leads | Flow |
|---|---|---|
| Smart lamp, IoT sensor, measuring device | Electronics | Board first → enclosure follows the board |
| Consumer product with a designed shape (remote, wearable) | Mechanical / industrial design | Shape first → the board must fit it |
| Robot, machine | Mechanics + kinematics | Assembly and motion → motors and drivers → firmware |

Both directions must work from the first integration milestone (M3).

## 1. Ownership

Every shared value has **exactly one owner** at a time. The other side receives it
as a constraint and cannot silently change it.

| Data | Owner |
|---|---|
| Board outline, thickness, mounting holes, keepout regions | **Either side** — set per project (`outline.owner`); mech-led products: MCAD; electronics-led: ECAD |
| Required positions of user-facing parts (connectors, buttons, LEDs, displays) | Either side, per interface |
| Component placement and component heights / 3D bodies | ECAD |
| Enclosure, brackets, fasteners | MCAD |
| Position of the board inside the assembly | MCAD |

Ownership can be handed over explicitly (recorded in history), e.g. the board outline
starts in ECAD for a prototype and moves to MCAD when industrial design begins.

## 2. Mechanical-led flow (shape first)

1. In CAD, the designer draws the **PCB envelope**: outline, mounting holes, keepout
   areas (with height limits), and required positions for user-facing parts.
2. The mech adapter reads these as entities; the hub creates interface objects
   `outline`, `mount`, `keepout`, and `placement`, all owned by MCAD.
3. In the PCB module they appear as **locked constraints**:
   - board outline and holes are read-only (owned elsewhere);
   - keepouts become rules, e.g. "inside this region, component height ≤ 3 mm";
   - components bound to a `placement` interface are pinned to their required position.
4. DRC reports violations of these constraints like any other rule.

## 3. Electronics-led flow (board first)

1. The PCB is laid out; ECAD owns outline and holes.
2. The PCB adapter exports **STEP** (board body, holes, component bodies from 3D models
   or extruded courtyards with heights) and the entities (`outline`, `mount`, connector
   frames).
3. The mech adapter places the board in the assembly as a **linked component**: when
   the PCB changes, the STEP is regenerated and the link updates.
4. The enclosure is created from the entities: bosses at `mount` holes, openings at
   connector `cutout`s, inner height from the `envelope` (tallest component + clearance).

## 4. Change proposals (after both sides exist)

Changes to shared values never overwrite the other side directly. They become
**change proposals**, similar to an engineering change order:

```
J1 (USB-C) moved +2.0 mm in x on the PCB
  → affects IF-usb (cutout), owned by: ECAD
  → enclosure opening "UsbOpening" no longer aligned
  [Accept: update enclosure]  [Reject: ask PCB to revert]  [Discuss]
```

- **Accept:** the receiving side is updated (by a person, or by the AI through the
  tool's MCP/CLI); the interface's `agreed` value becomes the new value.
- **Reject:** the proposal returns to the owner with a reason; the interface stays stale
  until resolved or overridden at the integration gate.
- A change by the side that does **not** own the value (e.g. CAD moves a hole that
  ECAD owns) is itself a proposal to the owner.
- Every proposal, decision, and reason is kept in project history.

Prior art worth studying (public documentation only): the **IDX** (EDMD) format
published by ProSTEP iViP for incremental ECAD–MCAD exchange with accept/reject of
proposed changes. Supporting it later would let projects exchange board data with
commercial tools. Older baseline format: **IDF**.

## 5. Automatic checks in the combined assembly

Run by the hub on the combined 3D model; results feed the **Integration** gate:

| Check | Example |
|---|---|
| Collision | Component body intersects an enclosure wall or rib |
| Height clearance | Tallest component vs lid, including a configurable margin |
| Connector access | Mating plug (from the part's data) fits through the opening |
| Fastener clearance | Screw head / washer vs copper and components around a mounting hole |
| Board fit | Outline vs enclosure pocket, with tolerance |

## 6. Interface types used here

| Type | Entities | Agreed values |
|---|---|---|
| `outline` | board outline ↔ enclosure pocket / envelope sketch | polygon, thickness, owner |
| `mount` | PCB hole ↔ boss/standoff/hole | position, diameter, fastener |
| `keepout` | PCB region ↔ CAD region | polygon, max height, side, kind (component/copper/via) |
| `placement` | PCB component ↔ CAD feature | position, rotation, side, tolerance |
| `cutout` | PCB connector/button/LED ↔ enclosure opening | center, size, facing side, clearance |
| `envelope` | component heights ↔ enclosure inner space | max height per region, clearance |

Manifest format: [03-project-manifest.md](03-project-manifest.md).

## 7. Same pattern in other domains

- **Embedded:** `pinmap` — ownership usually ECAD (schematic) but can be firmware-led
  (pin chosen in code, proposed back to the schematic).
- **Robotics:** `actuator` — the simulation's required torque/speed becomes a proposal
  to change the motor in the BOM and the driver on the PCB.
- **Simulation:** results never change designs directly; they create proposals or
  failed requirement checks.

## Needed from each module

| Module | Needs |
|---|---|
| PCB | STEP export; entity API (outline, holes, components with frames and heights); constraints from interfaces as locked rules; read-only outline when owned by MCAD |
| Mech (SolidCraft adapter) | Read sketches/features as entities by stable names; insert/update a linked STEP; create bosses/openings via MCP/CLI |
| Platform | Ownership, change proposals, history, combined-assembly checks |
| Hub | Proposal inbox, interface board, 3D combined view |

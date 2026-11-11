# 00 — Vision

## In one sentence

**Anyone can turn an idea into a working mechatronic product — in one open project,
with AI helping at every stage.**

## The problem

Building something like a smart lamp, a measuring device, or a robot arm today takes many
tools that do not know about each other: mechanical CAD, PCB EDA, a firmware IDE,
simulators, robotics tools. Data moves by hand through exports and imports. As a result:

- a change in one domain (the board moves 2 mm) goes unnoticed in another (the enclosure
  no longer fits) until a prototype fails;
- the BOM is scattered: electronics in the EDA tool, screws and housing in CAD, modules
  in someone's notes;
- beginners do not know where to start, and experts spend their time gluing data together;
- the integrated tools that exist are mostly closed, expensive, and online-only.

## Who it is for

| Group | Main need |
|---|---|
| Beginners, students, makers | Step-by-step guidance; describe what they want; ready-made templates |
| Engineers and hardware startups | Full editors, precise data, real fabrication, team collaboration |
| Educators | Example projects, works offline in a lab, free |
| Industry / small businesses | Self-hosted, data stays inside their own network |

Because the audience is **everyone**, the UX has two layers:

- **Guided mode (AI-first):** the user writes "I want an RGB bedside lamp I can control
  from my phone". The AI drafts requirements, proposes a system architecture, picks parts,
  and produces the schematic, PCB, enclosure, and firmware — all visible as a task list,
  pausing at every stage gate for approval.
- **Expert mode:** the full editor of each module, rules, CLI, scripting.

Both work on the same data; users can switch at any time.

## Product principles

1. **One project, many domains.** Requirements, BOM, versions, and stages span all modules.
2. **Cross-domain links are explicit** (interface objects), so changes are noticed.
3. **AI is a teammate, not a black box:** every AI step can be inspected, undone, and
   commented on; the AI uses the same tools as people.
4. **Open:** open-source code, documented file formats, usable by other tools.
5. **Local-first:** works offline; servers are optional for collaboration and heavy compute.
6. **Incremental:** every milestone produces something people can try.

## Non-goals (for now)

- Matching Fusion 360, SolidWorks, Altium, or ANSYS feature for feature.
- Rewriting good open-source tools (CAD kernels, FEA solvers, physics simulators).
- Full manufacturing/CAM (CNC toolpaths) — maybe later as an extra module.
- Enterprise PLM/ERP.

## What success looks like

- A beginner finishes a template project (e.g. the RGB lamp) all the way to PCB
  fabrication files, a printable enclosure, and flashable firmware — without touching
  another tool.
- An engineer can use any single part (e.g. only the PCB module) without being forced
  to use the rest.
- One change on the PCB raises the right warnings on the enclosure and in the firmware.

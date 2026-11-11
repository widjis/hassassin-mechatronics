# Research: SolidCraft as the mechanical tool

Date: 2026-11-11. Sources: shallow clone of `tangency-labs/solidcraft` (the old
`bherbruck/solidcraft` URL redirects there), `main` at the time of cloning, and the
browser build at `app.solidcraft.io` driven through its command API. Purpose: decide whether SolidCraft can carry
milestones M4 (agentic mechanical design) and M5 (two-way ECAD ↔ MCAD, [09-ecad-mcad.md](../09-ecad-mcad.md)).

Everything below marked **(docs)** comes from the project's own documentation and code;
**(tried)** was run in the browser build. Nothing was copied into our repository.

## Verdict

**Fit, with notes.** SolidCraft covers what M4 and M5 need through its CLI and MCP server,
its licensing is compatible with ours, and its design file is open JSON. The main gaps
are 2D drawings (in progress upstream) and live-linked STEP updates (we re-insert or
replace instead). See [Gaps and how we handle them](#gaps-and-how-we-handle-them).

## Licensing (checked in the repository)

- Code: **MIT OR Apache-2.0** (`LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`). GitHub's
  license detector shows only Apache-2.0; the repository files are authoritative.
- **Brand and Interface Licence** (`LICENSE-BRAND`) covers `crates/ui-kit`,
  `crates/look`, `crates/theme`, the name, logo, and app icon. Using or *linking* these
  from another program needs the maintainers' written permission. → We never link those
  crates and never ship anything that looks like SolidCraft.
- Geometry kernel: **solidcore** (vendored in `solidcore/`), a Rust port of NVIDIA's
  SMLib/SM_API from USD BRep, **Apache-2.0**. Own STEP reader written from the standard.
- Dependencies: 567 crates listed in `THIRD-PARTY-LICENSES.txt`, all permissive; the
  only GPL/LGPL mentions are alternatives in multi-licensed crates
  (e.g. `MIT OR Apache-2.0 OR LGPL-2.1-or-later`), which we may take under MIT/Apache.
- Their contribution rules forbid reading GPL/LGPL/AGPL code (OpenCascade, FreeCAD) —
  the same clean-room stance as ours.

## Capabilities relevant to us

| Need (M4, M5 and later) | SolidCraft | Source |
|---|---|---|
| Drive modeling from an agent or script | MCP server (`solidcraft-cli mcp`), headless or bridged to the running app; `execute`, `batch` with rollback, `describe_command` with JSON Schemas, `inspect_design`, `measure`, `body_topology`, `check_parameters`, `screenshot`, `export`, `open`, `save` | (docs) `docs/mcp.md` |
| Same commands in the browser | `window.solidcraft.execute / request / commands / inspect` | (docs) `docs/browser-api.md` |
| Build an electronics enclosure automatically | Official demo builds one over MCP: parametric box, shell, USB cut-out, four screw bosses, parameter change rebuilds | (docs) `examples/demo/enclosure_steps.json` |
| Enclosure features | `plastic.boss`, `plastic.lip`, `plastic.snap_fit`, `solid.rib`, shell, draft, holes, threads | (docs) `docs/features.md` |
| Import the PCB as 3D | STEP AP203/AP214/AP242 reader; each import becomes a component; STEP text is stored inside the design | (docs) `docs/step-import.md` |
| Stable references for interface objects | Named sketches (`sketch.create {name}`); faces named by role (`F2:side:l4`) and kept through edits; joints report persistent names; user parameters by name | (docs) `docs/mcp.md`, `docs/design/persistent-naming-proposal.md` |
| Store our ids inside the CAD file | `document.metadata.set` / `remove` — "other tools' keys" are kept in design properties | (docs) `docs/mcp.md` |
| Read the design from outside | `.sc` = ZIP with `container.json` + versioned `document.json` (`solidcraft/N`); plain JSON also accepted; blobs addressed by SHA-256 | (docs) `docs/design-container.md` |
| Linked components | Insert/Link/Relink design (`component.links`, `component.relink`) | (docs) `docs/features.md` |
| Assemblies, joints, motion | Components, joints, motion studies, contact sets, interference checks, mass properties | (docs) README, roadmap |
| Robotics | URDF export with per-link STL, mass, inertia | (docs) `docs/urdf.md` |
| Sheet metal | Flanges, hems, unfold, flat pattern, DXF | (docs) roadmap |
| 2D drawings | **In progress** (3 of 21 pieces; first geometry slice only) | (docs) `ROADMAP.md`, `docs/drawing-foundations.md` |
| Offline | Desktop app and CLI run locally; browser build also exists | (docs) README |

## Project health (at the time of checking)

- Very active: last push on the day of checking; ~15.5k GitHub stars (an older directory
  listing said ~500 — it grew fast).
- Strict engineering rules (never crash, every feature is a tested command, typed
  command contracts, crash reports).
- Risk: young and moving fast; command names and parameters can change. Their MCP
  server supports parameter contract checking (`--param-check strict`), which helps us
  detect breaking changes in CI.

## Hands-on results (tried)

A local release build of `solidcraft-cli` was **not completed**: the machine had too
little free disk space (the build was stopped by a guard at < 2 GB free; ~176 crates had
compiled). Instead, the same command set was exercised through the **browser build**
(`app.solidcraft.io`) with its `window.solidcraft` API, which uses the same command ids
and parameters as MCP `execute`.

| Test | Result |
|---|---|
| Command catalog | 570 commands; includes `file.insert_step`, `plastic.boss`, `plastic.lip`, `solid.shell`, `component.relink`, `file.export`, `document.metadata.set` |
| Replay the upstream enclosure demo (22 steps: parameters, named sketch `Outline`, extrude, fillet, shell, `USB port` cut, 4 × `plastic.boss`) | All steps succeeded |
| Change parameter `width` 80 → 100 mm | Rebuilt in ~1.8 s in the browser; body bbox became ±50 mm; no feature errors |
| Named sketches survive | `inspect` lists sketches `Outline`, `USB port` by name |
| Store our data in the design | `document.metadata.set` works when the value is an **object under a reverse-DNS key** (a plain string is refused); read back intact via `inspect` |
| Move a boss after creation | `timeline.edit {feature: "Boss1", set: {position: [...]}}` → 4 features recomputed, no errors |
| Rename a feature to an interface anchor | `timeline.rename` → `PCB_Mount_H1`; then addressable by that name |
| Boss position as an expression (`"width/2 - 10 mm"`) | **Refused** — `position` must be a literal point on a face |

Not tested yet (needs the CLI or file access): STEP insert of a real board, `.sc`
save/read from outside, URDF export, MCP over stdio, `--param-check strict`.

Implications for the adapter:
- Bosses and other point-placed features are **not parametric in position**: the mech
  adapter must recompute positions from PCB entities and apply them with
  `timeline.edit` by feature name.
- Our metadata key must be a reverse-DNS name chosen for the project (e.g.
  `org.hassassin.mechatronics`), holding an object.
- Name every feature that an interface points at (`PCB_Mount_H1`, `PCB_USB_Cutout`)
  right after creating it, and use names — not ordinal ids — everywhere.

## Gaps and how we handle them

| Gap | Plan |
|---|---|
| 2D drawings not ready | Not needed for M4/M5. Track upstream; contribute if we need it before it lands. |
| Inserted STEP is a copy, not a live link to the file | On PCB change, the mech adapter replaces the import (delete + `file.insert_step`, same component name and placement) and re-checks interfaces; ask upstream about a "reload STEP" command. |
| Persistent naming still has open issues upstream | Prefer **named sketches and user parameters** as interface anchors (`PCB_Envelope`, `pcb_width`) over picked faces; store our interface ids with `document.metadata.set`. |
| Commands change between releases | Pin a SolidCraft version per project (`toolVersion` in the manifest); run a contract test with `--param-check strict` against each new release. |
| Brand-licensed UI crates | Never link them; we integrate only through CLI, MCP, browser API, and files. |

## Embedding the web build (for M4)

The owner's target UX ([10-hub-ux.md](../10-hub-ux.md)) embeds SolidCraft's web build
next to the agent chat. Findings relevant to that:
- The browser build exposes `window.solidcraft` with the same command ids as MCP
  (tested above), so an agent bridge can drive it **if the page is on our origin**.
- The public `app.solidcraft.io` is a different origin and cannot be scripted from our
  page; self-hosting the unmodified web build (built with `trunk build --release`) is
  needed, or an upstream `postMessage` bridge.
- Distributing the unmodified app as SolidCraft is allowed; showing it *inside* another
  product may fall under `LICENSE-BRAND` §3a — ask the maintainers first.

## Recommendation

1. Keep ADR-3 (SolidCraft as the first mechanical tool); integrate through **MCP + CLI +
   `.sc`/STEP/URDF files**, never through its UI crates.
2. For M4/M5, anchor interfaces on **named sketches and parameters** in a template design
   (e.g. a `PCB_Envelope` sketch and `pcb_*` parameters) that our adapter creates and reads.
3. Do not fork yet. Open upstream issues for what we need (e.g. STEP reload), and fork only
   to prepare a pull request.

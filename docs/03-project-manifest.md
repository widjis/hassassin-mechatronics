# 03 — Project manifest (draft)

> Status: **draft v0**. Field names may still change until milestone M1 ships a JSON
> Schema in `spec/` and a reference implementation in `packages/platform`.

A project is a folder with a `hassassin.project` file (JSON, UTF-8) at its root. All
paths are relative to that folder and use `/`.

## Example

```json
{
  "format": "hassassin.project",
  "version": 0,
  "id": "prj_7f3a9c",
  "name": "RGB bedside lamp",
  "created": "2026-11-11T08:00:00Z",

  "requirements": {
    "goal": "Bedside lamp with adjustable RGB light, controlled from a phone over BLE",
    "supply": "USB-C 5 V",
    "current": "≤ 1 A",
    "size": "≤ 90 mm diameter",
    "environment": "Indoor, 10–40 °C",
    "quantity": "5 prototypes",
    "items": [
      { "id": "REQ-1", "text": "LED current ≤ 20 mA per channel", "verifiedBy": ["sim:led-current"] }
    ]
  },

  "documents": [
    { "id": "pcb",  "kind": "pcb",      "path": "electronics/main.hpcb", "tool": "hassassin-pcb" },
    { "id": "case", "kind": "mech",     "path": "mechanical/case.sc",    "tool": "solidcraft", "toolVersion": "pinned" },
    { "id": "fw",   "kind": "embedded", "path": "firmware/",             "tool": "platformio" },
    { "id": "sim1", "kind": "sim",      "path": "simulation/led.json",   "tool": "ngspice" }
  ],

  "interfaces": [
    {
      "id": "IF-mount-1",
      "type": "mount",
      "a": { "doc": "pcb",  "entity": "hole:H1" },
      "b": { "doc": "case", "entity": "feature:Boss1" },
      "agreed": { "position": [3.5, 3.5], "diameter": 3.2, "unit": "mm" },
      "seen": { "a": "rev:41", "b": "rev:12" },
      "status": "ok"
    },
    {
      "id": "IF-usb",
      "type": "cutout",
      "a": { "doc": "pcb",  "entity": "component:J1" },
      "b": { "doc": "case", "entity": "feature:UsbOpening" },
      "agreed": { "center": [0, -21.0, 1.8], "size": [9.2, 3.4], "side": "-y" },
      "status": "stale",
      "change": { "by": "pcb", "summary": "J1 moved 2.0 mm in +x", "at": "2026-11-12T10:15:00Z" }
    },
    {
      "id": "IF-pins",
      "type": "pinmap",
      "a": { "doc": "pcb", "entity": "component:U1" },
      "b": { "doc": "fw",  "entity": "file:include/board_pins.h" },
      "status": "ok"
    }
  ],

  "bom": { "generatedFrom": ["pcb", "case"], "extra": [
    { "item": "M3x6 screw", "qty": 4, "source": "manual" }
  ] },

  "stages": {
    "overrides": { "verification": "Prototype only; thermal test done by hand" }
  },

  "releases": [
    { "name": "Proto A", "at": "2026-11-20T09:00:00Z", "snapshot": ".hassassin/releases/proto-a" }
  ]
}
```

## Fields

| Field | Meaning |
|---|---|
| `format`, `version` | Always `"hassassin.project"`; integer version of the manifest format |
| `id`, `name`, `created` | Stable project id, display name, creation time (ISO 8601) |
| `requirements` | Structured requirements: standard fields + `items` with ids that other objects can trace to |
| `documents[]` | One entry per document: `id` (unique in project), `kind`, `path`, `tool`, optional `toolVersion` |
| `interfaces[]` | Interface objects (below) |
| `bom` | Which documents feed the combined BOM, plus manual extra lines |
| `stages` | Gate overrides with reasons; gate state itself is computed, not stored |
| `releases[]` | Named snapshots of all documents + manifest |

## Interface objects

| Field | Meaning |
|---|---|
| `id` | Unique in the project |
| `type` | `mount`, `cutout`, `envelope`, `pinmap`, `actuator`, `sensor`, … (extensible) |
| `a`, `b` | Entity references `{doc, entity}`; entity ids are defined by each adapter (`hole:H1`, `component:J1`, `feature:Boss1`, `joint:shoulder`) |
| `agreed` | The values both sides agreed on (units explicit) |
| `seen` | Document revision of each side when last checked |
| `status` | `ok`, `stale` (one side changed), `conflict` (both changed), `broken` (entity missing) |
| `change` | When not `ok`: which side changed, summary, time |

Rules:
- **Status is recomputed** by adapters; a stored `status` is a cache, not truth.
- Resolving a stale interface = either update the other side (via its adapter/tool) or
  accept the new value as `agreed`. Both are recorded in history.
- Unknown `type`s must be preserved by every tool (forward compatibility).

## Open questions

- Should requirements live in the manifest or in a separate `requirements.json`?
- Entity id stability: how to keep ids stable when a CAD feature is rebuilt or a part is renamed.
- How collaboration (operation sync) applies to the manifest itself.

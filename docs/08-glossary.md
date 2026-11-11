# 08 — Glossary

| Term | Meaning |
|---|---|
| **Adapter** | Small module that connects one tool to the platform: reads entities and BOM, exports, applies approved changes |
| **Agent / AI orchestrator** | The AI that works across modules through MCP tools, with a visible task list |
| **BOM** | Bill of materials: every part to buy or make, across all domains |
| **Change proposal** | A proposed new value for an interface, accepted or rejected by the receiving side |
| **Command layer** | The rule that every document change is a small invertible operation |
| **ECO** | Engineering change order: a recorded change and what it affects |
| **Entity** | Something inside a document that interfaces can point at (hole, connector, pin, joint) |
| **Federation** | Many independent tools joined through shared project data, instead of one big program |
| **Gate** | A check at the end of a stage (e.g. "DRC clean"); can be overridden with a reason |
| **Hub** | The main app: project home, stages, BOM, interfaces, AI panel |
| **IDX / EDMD** | Public ProSTEP iViP format for incremental ECAD–MCAD exchange with accept/reject of changes |
| **Interface object** | A link between entities in two documents with an agreed value and a status |
| **Local-first** | Works fully offline; servers are optional |
| **Keepout** | A region where components, copper, or vias are restricted (often with a height limit) |
| **Manifest** | The `hassassin.project` file describing a project |
| **MCAD / ECAD** | Mechanical CAD / electronic CAD (schematic and PCB) |
| **MCP** | Model Context Protocol: the standard way AI agents call tools |
| **Owner (of an interface)** | The one side allowed to change an interface's agreed value |
| **Stage** | A project step: requirements → architecture → design → integration → verification → production |
| **Stale** | An interface whose one side changed since the agreed value |
| **URDF / MJCF** | Robot description formats (links, joints, inertia) |
| **STEP** | Neutral 3D CAD exchange format |

---
type: domain
title: "ECS Registry"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - ecs
status: seed
page_count: 0
---

# ECS Registry

> Live registry of all Entities and Components in the project. The ground truth for what exists in the game world.

## Entities

_See [[ecs/entities/_index]]_

| Entity | Components | Status |
|--------|-----------|--------|
| _None yet_ | — | — |

## Components

_See [[ecs/components/_index]]_

| Component | Data Fields | Used By |
|-----------|------------|---------|
| _None yet_ | — | — |

---

## ECS Architecture

This vault uses the [[ECS]] pattern. See that page for the full theory, pros/cons, and code skeletons.

**Component-to-Entity Linkage Rule:**
Every Entity note must contain a Bases query pulling from its constituent Component notes. Format:

```
## Components

```dataview
TABLE component_type, data_fields
FROM "wiki/ecs/components"
WHERE contains(used_by_entities, this.file.name)
```
```

*(Replace with Bases query once dashboard.base pattern is confirmed for your Obsidian version.)*

## System Registry

| System | Reads | Writes | Archetype |
|--------|-------|--------|-----------|
| _None yet_ | — | — | — |

## Related
- [[ECS]] — pattern documentation
- [[wiki/game-design/technical-architecture/_index]] — system architecture
- [[wiki/patterns/_index]] — pattern library

---
type: domain
title: "ECS Entities Index"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - ecs
  - entities
status: seed
page_count: 0
---

# ECS Entities

> All game entities. Each entity note declares which components it requires and queries component data via Bases.

## All Entities

_None yet. Template: `_templates/entity-ecs.md`_

---

## Entity Template Structure

```
Entity: [Name]
├── Components: [list of component notes]
├── Archetypes: [variants — e.g., Player, Enemy, NPC]
├── Systems that operate on it: [list]
└── Bases query: pulls live data from component notes
```

## Archetype Map

| Entity | Required Components | Optional Components |
|--------|--------------------|--------------------|
| _None yet_ | — | — |

## Related
- [[wiki/ecs/components/_index]] — component definitions
- [[wiki/ecs/_index]] — ECS registry root

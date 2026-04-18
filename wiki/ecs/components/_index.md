---
type: domain
title: "ECS Components Index"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - ecs
  - components
status: seed
page_count: 0
---

# ECS Components

> All component definitions. Pure data — no methods. Components are the atoms of the ECS system.

## All Components

_None yet. Template: `_templates/component-ecs.md`_

---

## Component Naming Convention

- `[Domain][Concept]Component` → e.g., `MovementSpeedComponent`, `HealthCurrentComponent`
- Or short names for widely-used components: `Health`, `Transform`, `Velocity`

## Component Categories

| Category | Examples |
|----------|---------|
| Identity | `TagPlayer`, `TagEnemy`, `TeamComponent` |
| Physics | `PositionComponent`, `VelocityComponent`, `MassComponent` |
| Combat | `HealthComponent`, `DamageComponent`, `ArmorComponent` |
| AI | `PatrolComponent`, `AggroComponent`, `NavigationComponent` |
| Visual | `SpriteComponent`, `AnimationStateComponent` |
| Audio | `AudioTriggerComponent` |
| Meta | `DeadTag`, `DirtyTag`, `SpawnedTag` |

## Related
- [[wiki/ecs/entities/_index]] — entities that use these components
- [[wiki/ecs/_index]] — registry root
- [[ECS]] — pattern documentation

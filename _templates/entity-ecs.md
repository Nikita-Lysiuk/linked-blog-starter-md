---
type: ecs-entity
title: "entity-ecs"
created: 2026-04-18
updated: 2026-04-18
tags:
  - ecs
  - entity
status: seed
archetype_variants: []
required_components: []
optional_components: []
related: []
---

# entity-ecs (Entity)

**Archetype:** [Base archetype name]
**Variants:** [e.g., Player, Enemy, NPC, Projectile]

---

## Component Manifest

> Required components — this entity cannot exist without these.

| Component | Purpose | Link |
|-----------|---------|------|
| | | [[wiki/ecs/components/]] |

> Optional components — conditionally attached.

| Component | Condition | Link |
|-----------|-----------|------|
| | | [[wiki/ecs/components/]] |

---

## Live Component Data

> Bases query: pulls current data from component notes.
> Enable Obsidian Bases (Core Plugins) to activate.

```dataview
TABLE component_type, data_fields, status
FROM "wiki/ecs/components"
WHERE contains(used_by_entities, "entity-ecs")
```

---

## Systems That Operate on This Entity

| System | Operation | Reads | Writes |
|--------|-----------|-------|--------|
| | | | |

---

## Archetype Variants

Describe each variant and which components differ:

### [Variant Name]
- **Differs:** extra/removed components
- **Behavior:** 

---

## Code Reference

```csharp
// Unity DOTS — Archetype definition
EntityArchetype entity-ecsArchetype = entityManager.CreateArchetype(
    typeof(/* Component1 */),
    typeof(/* Component2 */)
);
```

```cpp
// entt — entity creation
auto entity = registry.create();
registry.emplace</* Component1 */>(entity);
registry.emplace</* Component2 */>(entity);
```

## Related
- [[wiki/ecs/_index]]
- [[wiki/ecs/components/_index]]
- [[ECS]]

---
type: ecs-component
title: "component-ecs"
created: 2026-04-18
updated: 2026-04-18
tags:
  - ecs
  - component
status: seed
component_type: ""
used_by_entities: []
related: []
---

# component-ecs (Component)

**Category:** `Identity` / `Physics` / `Combat` / `AI` / `Visual` / `Audio` / `Meta`
**Type:** Pure data — no methods.

---

## Data Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| | | | |

---

## Code Definition

```csharp
// Unity DOTS — IComponentData
public struct component-ecs : IComponentData
{
    // Add fields here
}
```

```cpp
// C++ / entt — plain struct
struct component-ecs {
    // Add fields here
};
```

---

## Used By Entities

```dataview
TABLE status
FROM "wiki/ecs/entities"
WHERE contains(required_components, "component-ecs")
   OR contains(optional_components, "component-ecs")
```

---

## Systems That Read/Write This Component

| System | Access | Purpose |
|--------|--------|---------|
| | Read / Write | |

---

## Design Notes

> Why does this component exist as a separate data struct? What would break if it were merged into another component?

## Related
- [[wiki/ecs/_index]]
- [[wiki/ecs/entities/_index]]
- [[ECS]]

---
type: concept
title: "Entity-Component-System (ECS) Pattern"
created: 2026-04-18
updated: 2026-04-18
tags:
  - pattern
  - architectural
  - ecs
  - performance
status: seed
complexity: advanced
domain: software-architecture
related:
  - "[[Observer]]"
  - "[[Singleton]]"
  - "[[patterns/_index]]"
  - "[[ecs/_index]]"
sources: []
---

# Entity-Component-System (ECS)

**Category:** Architectural
**Intent:** Decouple data (Components) from behavior (Systems), with Entities as pure IDs that bind components together. Enables data-oriented design, cache-friendly memory layout, and massive parallelism.

---

## Structure

```
Entity    → just an ID (uint32 or handle)
Component → pure data struct, no methods
System    → logic that queries and operates on components

World
├── EntityRegistry: Map<EntityID, ComponentMask>
├── ComponentArrays: SoA (Structure of Arrays) per component type
└── SystemList: ordered list of Systems

Per frame:
  for each System:
      query(ComponentMask) → EntitySet
      for each Entity in EntitySet:
          read/write Components
```

---

## Pros

- **Cache efficiency:** SoA layout — all `HealthComponent` data contiguous in memory. Traditional OOP scatters data across heap.
- **Composition over inheritance:** Add/remove components at runtime. No deep inheritance trees.
- **Parallelism:** Systems with non-overlapping component access run in parallel (Unity Jobs, Bevy systems)
- **Testability:** Systems are pure functions on data — trivial to unit test without a full game world
- **Scalability:** 10,000 entities with the same components = same performance profile per entity

## Cons

> [!contradiction] ECS vs Game Logic Expressiveness
> ECS excels at data-heavy simulations (10k soldiers, physics, particles). For complex, unique game logic with many special cases (boss AI, dialogue trees, cutscenes), ECS can be more verbose than a well-structured OOP approach. Consider a hybrid.

- Steeper learning curve than OOP GameObjects
- "Reactive" logic (do X when Y changes) requires event/structural-change systems
- Debugging: component data is decoupled from behavior — harder to trace "why did this entity die?"
- Unity DOTS ECS has significant boilerplate (IComponentData, ISystem, IJobEntity)
- Circular component dependencies require careful architectural separation

## Game Context

**Use when:**
- Large numbers of similar entities (enemies, projectiles, particles, crowd AI)
- Performance is critical — you're hitting CPU bottlenecks with classical GameObjects
- You want deterministic, replayable, or networked simulation
- Team is comfortable with data-oriented design

**Avoid when (or use hybrid):**
- Small indie project where DOTS overhead isn't warranted
- Unique, complex entities with lots of bespoke logic (use classical OOP + components)
- Heavy use of coroutines, async/await patterns not yet fully supported in DOTS

**Engine notes:**
- **Unity DOTS:** `IComponentData`, `ISystem`, `IJobEntity`, `Burst` compiler, `EntityCommandBuffer`
- **Godot:** No native ECS — use third-party (limboai, flecs-godot) or implement manually
- **Unreal:** Mass Entity system (UE5+) — AAA-grade ECS built-in
- **Custom/C++:** `entt` library is the gold standard (header-only, extremely fast)

---

## Code Skeleton (C# — Unity DOTS)

```csharp
// Component — pure data, no methods
public struct HealthComponent : IComponentData
{
    public int Current;
    public int Max;
}

public struct SpeedComponent : IComponentData
{
    public float Value;
}

// System — logic that operates on components
[BurstCompile]
public partial struct MovementSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        float dt = SystemAPI.Time.DeltaTime;

        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<SpeedComponent>>())
        {
            transform.ValueRW.Position += math.forward() * speed.ValueRO.Value * dt;
        }
    }
}

// Creating an entity
public class EntitySpawner : MonoBehaviour
{
    public void SpawnEnemy(EntityManager em)
    {
        Entity enemy = em.CreateEntity(
            typeof(HealthComponent),
            typeof(SpeedComponent)
        );
        em.SetComponentData(enemy, new HealthComponent { Current = 100, Max = 100 });
        em.SetComponentData(enemy, new SpeedComponent { Value = 5f });
    }
}
```

```cpp
// C++ with entt (https://github.com/skypjack/entt)
#include <entt/entt.hpp>

struct Health { int current; int max; };
struct Speed  { float value; };
struct Position { float x, y, z; };

// Create world
entt::registry registry;

// Spawn entity
auto enemy = registry.create();
registry.emplace<Health>(enemy, 100, 100);
registry.emplace<Speed>(enemy, 5.0f);
registry.emplace<Position>(enemy, 0.f, 0.f, 0.f);

// System — movement
auto MovementSystem = [&](float dt) {
    auto view = registry.view<Position, const Speed>();
    view.each([dt](Position& pos, const Speed& spd) {
        pos.z += spd.value * dt;
    });
};

// Game loop
MovementSystem(deltaTime);
```

---

## Related Patterns

- **[[Observer]]** — ECS handles events as structural changes (add `DeadTag` component) rather than callbacks
- **[[Singleton]]** — avoid; use `SystemAPI.GetSingleton<T>()` or World context instead
- **Component** (GoF) — ECS component is a pure-data variant of the GoF Component pattern
- **Data Locality** (Game Programming Patterns) — ECS is the canonical implementation

> [!key-insight] Start Here
> ECS is the architectural backbone of this vault. Every mechanic, every entity, every system maps to this pattern. The `wiki/ecs/` folder is the live registry of all entities and components in the project.

## See Also
- [[ecs/_index]] — live ECS entity/component registry
- [[game-design/technical-architecture/_index]] — system diagrams

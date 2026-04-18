---
type: domain
title: "Pattern Library Index"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - patterns
status: developing
page_count: 3
---

# Pattern Library

> Software design patterns with Pros, Cons, Game Context, and C#/C++ code skeletons.

## Seeded Patterns

| Pattern | Category | Status | Game Use |
|---------|----------|--------|----------|
| [[Singleton]] | Creational | seed | GameManager, AudioSystem, InputHandler |
| [[Observer]] | Behavioral | seed | Events, UI updates, achievement triggers |
| [[ECS]] | Architectural | seed | Core game loop architecture |

## Adding a New Pattern

Use `_templates/pattern.md`. Every pattern must include:
1. **Intent** — one-sentence description
2. **Structure** — UML-style pseudocode
3. **Pros** — concrete advantages in game context
4. **Cons** — concrete pitfalls in game context
5. **Game Context** — when to use, when to avoid, engine notes
6. **Code Skeleton** — C# and/or C++ minimum viable implementation
7. **Related Patterns** — what it competes with or complements

## Pattern Categories

| Category | Patterns |
|----------|---------|
| Creational | [[Singleton]], Factory, Object Pool |
| Structural | Flyweight, Decorator, Facade |
| Behavioral | [[Observer]], Command, State, Strategy |
| Architectural | [[ECS]], Service Locator, Event Bus, Component |
| Concurrency | Job System, Double Buffer, Dirty Flag |

## Related
- [[wiki/ecs/_index]] — ECS pattern in full detail
- [[wiki/decisions/_index]] — ADRs that chose between patterns
- [[wiki/comparisons/_index]] — pattern vs pattern analyses

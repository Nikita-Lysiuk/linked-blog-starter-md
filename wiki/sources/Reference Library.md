---
type: source
title: "Reference Library"
created: 2026-04-18
updated: 2026-04-18
tags:
  - source
  - library
  - reference
status: developing
source_type: book
related:
  - "[[sources/_index]]"
  - "[[concepts/Collisions]]"
  - "[[patterns/_index]]"
---

# Reference Library

> Central catalog of all books, papers, and reference materials. Link here from concept notes with chapter/page citations.

---

## Game Development & Engine Programming

| # | Title | Author | Year | Key Topics | Notes |
|---|-------|--------|------|------------|-------|
| 1 | **Game Programming Patterns** | Robert Nystrom | 2014 | Design patterns in games, Command, Observer, State, ECS precursors | Free online at gameprogrammingpatterns.com |
| 2 | **Game Engine Architecture** (3rd ed.) | Jason Gregory | 2018 | Engine layers, rendering pipeline, physics integration, memory | Ch. 12: Collision & Physics |
| 3 | **Real-Time Collision Detection** | Christer Ericson | 2005 | AABB, OBB, Sphere, SAT, GJK, BVH, spatial partitioning | **The** collision reference — see [[concepts/Collisions]] |
| 4 | **Physics for Game Developers** (2nd ed.) | David M. Bourg & Bryan Bywalec | 2013 | Rigid body dynamics, collision response, friction, joints | Ch. 3–5: Collision detection & resolution |
| 5 | **3D Math Primer for Graphics and Game Dev** (2nd ed.) | Fletcher Dunn & Ian Parberry | 2011 | Vectors, matrices, quaternions, geometric tests | Ch. 9: Geometric tests (ray-AABB, sphere-sphere) |
| 6 | **Real-Time Rendering** (4th ed.) | Akenine-Möller et al. | 2018 | Rendering pipeline, spatial data structures, BVH | Ch. 22: Intersection test methods |

---

## Software Architecture & Patterns

| # | Title | Author | Year | Key Topics | Notes |
|---|-------|--------|------|------------|-------|
| 7 | **Clean Architecture** | Robert C. Martin | 2017 | SOLID, dependency inversion, use cases, boundaries | Relevant for `IAbilitySystem` interface design |
| 8 | **Design Patterns: Elements of Reusable OO Software** | GoF (Gang of Four) | 1994 | 23 classic patterns: Singleton, Observer, Strategy, etc. | Foundation for [[patterns/_index]] |
| 9 | **Data-Oriented Design** | Richard Fabian | 2018 | DOD vs OOP, cache efficiency, SoA, ECS | Free online at dataorienteddesign.com. Foundation for [[ECS]] |

---

## Unreal Engine & C++

| # | Title | Author | Year | Key Topics | Notes |
|---|-------|--------|------|------------|-------|
| 10 | **Unreal Engine 5 Game Development with C++** | Packt | 2023 | UE5 C++ APIs, GAS overview, component architecture | Chapter on GAS relevant to [[Ability System Interface]] |
| 11 | **Learning C++ by Building Games with UE4** | William Sherif | 2015 | UE4/5 C++ patterns, component system, actor model | Older but fundamentals still valid |

---

## Academic Papers & Online Resources

| # | Title | Author | Year | URL | Key Topics |
|---|-------|--------|------|-----|------------|
| P1 | Separating Axis Theorem (SAT) tutorial | dyn4j.org | 2010 | [dyn4j.org/2010/01/sat](https://dyn4j.org/2010/01/sat/) | SAT walkthrough — see [[concepts/Collisions]] |
| P2 | GJK Algorithm explanation | Casey Muratori | 2006 | [mollyrocket.com/849](https://mollyrocket.com/849) | GJK video series |
| P3 | Wikipedia: Collision Detection | Wikipedia | — | [en.wikipedia.org/wiki/Collision_detection](https://en.wikipedia.org/wiki/Collision_detection) | Overview, AABB, BSP trees |

---

## How to Cite from This Library

In concept notes, reference as:
```
> Source: [[Reference Library]] — [Book #N], Ch. X, p. YYY
```

Example: `> Source: [[Reference Library]] — [Book #3 — Ericson], Ch. 4, p. 77 (AABB vs AABB)`

## Related
- [[sources/_index]]
- [[concepts/Collisions]] — primary consumer of this library
- [[patterns/_index]] — GoF, Game Programming Patterns

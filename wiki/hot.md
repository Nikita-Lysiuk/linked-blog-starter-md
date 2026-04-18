---
type: meta
title: "Hot Cache"
updated: 2026-04-18T14:00:00
tags:
  - meta
  - hot-cache
---

# Recent Context

## Last Updated
2026-04-18. Deep vault refactor complete. First real content created.

## Current Sprint
**Project Persival** — Ability System foundation.
**Active task:** [[Universal Ability System Interface]] — `IAbilitySystem` C++ interface + `UAbilityManagerComponent` designed. Awaiting prototype.

**Learning Campus** — [[Collisions]] theory page complete. Collision Sandbox project staged (3 phases).

## Active Technical Roadblocks

1. **Disguise Steal — Contradict unresolved:** Alert-state behavior during disguise (does combat-alert break disguise?) not yet decided. Filed in mechanic note. Needs ADR.

2. **IAbilitySystem — Interrupt priority:** No priority system for simultaneous ability activations. Works for now; must resolve before second ability is implemented.

## Key Recent Facts
- `IAbilitySystem` interface designed with `ExecuteAbility`, `CancelAbility`, `GetCooldownRemaining`, `IsOnCooldown`
- `UAbilityManagerComponent` manages ability slots — direct activation, no queue
- Disguise Steal: steals guard's `USkeletalMeshComponent` + materials, AI perception override, 60s duration, 30s cooldown
- Reference Library: 11 books + 3 papers cataloged — primary collision reference is Ericson (Book #3)
- Collisions concept covers AABB, Sphere, SAT, GJK with C++ code skeletons and algorithm comparison table

## Recent Changes
- Created: [[Disguise Steal]], [[Universal Ability System Interface]], [[Ability System Interface]]
- Created: [[sources/Reference Library]], [[concepts/Collisions]]
- Created: [[meta/kanban-persival]], [[meta/kanban-campus]], [[meta/Dashboard]]
- Updated: [[wiki/index]] — Kanban embeds added

## Active Threads
- Next: prototype `UDisguiseAbility` in UE5 — implement `ExecuteAbility_Implementation`
- Next: flesh out `wiki/game-design/loop-logic/` — core loop for Persival not yet defined
- Next: complete Collision Sandbox Stage 1 (math setup)
- Open: define Player entity in `wiki/ecs/entities/`
- Open: define Guard entity in `wiki/ecs/entities/`

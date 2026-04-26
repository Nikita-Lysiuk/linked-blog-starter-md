---
type: decision
title: 'Ability Architecture — Why BT-Driven State, Not Custom Ability Manager'
created: '2026-04-18'
updated: '2026-04-27'
tags:
  - technical-architecture
  - adr
  - persival
  - ability-system
  - sprint
status: decided
priority: 1
related:
  - '[[Ability System Interface]]'
  - '[[AI System Architecture]]'
  - '[[FaceSteal]]'
  - '[[Mind Copy]]'
  - '[[game-design/technical-architecture/_index]]'
---

# Ability Architecture — Why BT-Driven State, Not Custom Ability Manager

**Project:** Persival
**Status:** Decided and implemented
**Replaces:** Earlier proposal for `IAbilitySystem` + `UAbilityManagerComponent` (discarded — never built)

---

## Problem

Two player abilities (P1 FaceSteal, P2 MindCopy) need to affect AI characters without creating tight coupling between player code and AI class hierarchy. The AI needs complex, time-sequenced behaviour after an ability fires (disorientation wake-up → state change → expiry restore) — behaviour that must integrate with UE5's Behavior Tree.

---

## Options Evaluated

### Option A: `IAbilitySystem` + `UAbilityManagerComponent` (rejected)
Custom `ExecuteAbility / CancelAbility` interface on a manager component attached to the player.

**Problems:**
- Ability effects need time-sequenced BT integration. An `ExecuteAbility` call can't cleanly hand off to BT tasks.
- Duplicates BT's own state management — two state machines fighting each other.
- Cooldown/duration logic belongs in config (DataAsset), not in an ability object.

### Option B: Unreal GAS (rejected for now)
Gameplay Ability System — industry standard, replication built-in.

**Problems:**
- Large footprint: ASC on every NPC, Attribute Sets, Gameplay Effects — overkill for 2 abilities at prototype stage.
- Steep learning curve for a small team.
- GAS prediction wasted work (local splitscreen, no online yet).

### Option C: BT-Driven State + Interface Boundary (chosen)
Abilities change `ConsciousnessState` via `UPR_AIMemoryComponent`. The BT responds to state changes natively. All time-sequenced behaviour lives in BT tasks and services.

**Advantages:**
- BT is already the AI's control system — no second state machine.
- `DisorientDuration`, `MindReplaceDuration` live in `UPR_EnemyConfig` DataAssets — tunable per enemy.
- `IPR_AIAbilityTarget` interface isolates player code from AI class hierarchy.
- Replication is simple: replicate `ConsciousnessState`, keep snapshots server-only.

---

## Implemented Architecture

```
Player Ability Components (IPR_PlayerAbility)
  │
  ├── UPR_FaceStealComponent (P1 — PR-87)
  │     Uses IPR_AIAbilityTarget to:
  │       CanBeTargetedByAbility() → gate
  │       GetMemoryComponent() → SetConsciousnessState(Frozen)
  │       Cast to APR_BaseCharacter → RuntimeTags copy
  │       BeginDisorientation() → on deactivate
  │
  └── UPR_TelepathyComponent (P2 — pending PR-89/90/91)
        Uses IPR_AIAbilityTarget to:
          CanBeTargetedByAbility() → gate
          GetMemoryComponent() → TakeSnapshot / transfer
          BeginDisorientation() → on paste expiry

IPR_AIAbilityTarget (query + access boundary — never ability-specific)
  ├── CanBeTargetedByAbility()       → immunity gate
  ├── GetMemoryComponent()           → UPR_AIMemoryComponent access
  ├── BeginDisorientation()          → post-ability wake-up trigger
  └── GetCurrentConsciousnessState() → state query

UPR_AIMemoryComponent
  ├── SetConsciousnessState(State)   → replicates, broadcasts delegate
  ├── TakeSnapshot(BB)               → P2 Copy
  ├── TakeOwnStateSnapshot(BB)       → Paste B pre-save
  └── RestoreFromSnapshot / RestoreFromOwnStateSnapshot

APR_AIController
  ├── RestoreFromSnapshot()          → BT task callback, touches BB
  ├── RestoreFromOwnStateSnapshot()  → BT task callback, touches BB
  └── ApplySnapshotToBB()            → private BB write helper

BT_AIBase (priority selector)
  ├── ConsciousnessState != Normal → BT_ControlledByPlayer subtree
  │     ├── BTTask_PR_HandleFrozen         (P1 — holds indefinitely)
  │     ├── BTTask_PR_Disorient            (wake-up after any ability deactivation)
  │     └── [Replaced: standard BT with donor BB state]
  └── Standard patrol / alert / investigate branches
```

---

## Review Trigger

Revisit if:
- **Online multiplayer** becomes a goal → GAS prediction valuable; this architecture maps cleanly onto an adapter layer
- **More than ~6 abilities** planned → evaluate BT task proliferation
- **Ability combos / queuing** needed → add explicit queue layer between input and `SetConsciousnessState`

---

## Related
- [[Ability System Interface]] — `IPR_AIAbilityTarget` interface definition
- [[AI System Architecture]] — full system map
- [[Consciousness State Machine]] — state transitions in detail
- [[FaceSteal]], [[Mind Copy]] — the two abilities this architecture serves

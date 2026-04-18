---
type: decision
title: 'Ability Architecture — Why BT-Driven State, Not Custom Ability Manager'
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
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
  - '[[Disguise Steal]]'
  - '[[Mind Copy]]'
  - '[[game-design/technical-architecture/_index]]'
---

# Ability Architecture — Why BT-Driven State, Not Custom Ability Manager

**Project:** Persival
**Status:** Decided and implemented
**Replaces:** Earlier proposal for `IAbilitySystem` + `UAbilityManagerComponent` (discarded — never built)

> [!correction] The previous version of this note proposed `IAbilitySystem::ExecuteAbility/CancelAbility` and `UAbilityManagerComponent`. **None of that was implemented.** The actual architecture is described here.

---

## Problem

Two player abilities (P1 FaceSteal, P2 MindCopy) need to affect AI characters without creating tight coupling between player code and AI class hierarchy. Additionally, the AI needs complex, time-sequenced behavior after an ability fires (disorientation freeze → behavior change → expiry restore) — behavior that must integrate with UE5's Behavior Tree.

---

## Options Evaluated

### Option A: `IAbilitySystem` + `UAbilityManagerComponent` (rejected)
Custom `ExecuteAbility / CancelAbility` interface on a manager component attached to the player.

**Problems:**
- Ability effects need time-sequenced BT integration (disorientation → replaced state → expiry). An `ExecuteAbility` call can't cleanly hand off to BT tasks.
- Duplicates BT's own state management — two state machines fighting each other.
- Cooldown/duration logic belongs in config (DataAsset), not in an ability object.

### Option B: Unreal GAS (rejected for now)
Gameplay Ability System — industry standard, replication built-in.

**Problems:**
- Large footprint: ASC on every NPC, Attribute Sets, Gameplay Effects — overkill for 2 abilities.
- Steep learning curve for a 3-person team at prototype stage.
- GAS prediction system would be wasted work (local splitscreen, no online yet).

### Option C: BT-Driven State + Interface Boundary (chosen)
Abilities change `ConsciousnessState` via `UPR_AIMemoryComponent`. The BT responds to state changes natively. All time-sequenced behavior lives in BT tasks and services.

**Advantages:**
- BT is already the AI's control system — no second state machine.
- `DisorientDuration`, `MindReplaceDuration` live in `UPR_EnemyConfig` DataAssets — tunable per enemy type.
- `IPR_AIAbilityTarget` interface isolates player code from AI class hierarchy.
- Replication is simple: replicate `ConsciousnessState`, keep snapshots server-only.

---

## Implemented Architecture

```
Player Ability Code
  │ (uses only IPR_AIAbilityTarget — no concrete casts)
  ▼
IPR_AIAbilityTarget
  ├── CanBeTargetedByAbility()       → gate (mechanical enemies immune)
  ├── GetMemoryComponent()           → UPR_AIMemoryComponent
  └── GetCurrentConsciousnessState() → EAIConsciousnessState

UPR_AIMemoryComponent
  ├── SetConsciousnessState(State)   → replicates, broadcasts delegate
  ├── TakeSnapshot(BB)               → P2 Copy
  ├── TakeOwnStateSnapshot(BB)       → Paste B pre-save
  └── Restore* methods               → called from controller

APR_AIController
  ├── ApplyPasteA()                  → sets Overwritten + BB timing keys
  └── ApplyPasteB()                  → sets Overwritten + Replaced timing

BT_AIBase (priority selector)
  ├── ConsciousnessState != Normal → BT_ControlledByPlayer subtree
  │     ├── BTTask_PR_HandleFrozen         (P1)
  │     ├── BTTask_PR_HandleOverwritten    (P2 transition)
  │     └── [Replaced: standard BT with donor BB state]
  └── Standard patrol / alert / investigate branches
```

---

## Review Trigger

Revisit this decision if:
- **Online multiplayer** becomes a goal → GAS prediction becomes valuable; this architecture maps cleanly onto an adapter layer
- **More than ~6 abilities** are planned → evaluate if BT task proliferation is acceptable
- **Ability combos / queuing** are needed → add explicit queue layer between player input and `SetConsciousnessState`

---

## Related
- [[Ability System Interface]] — `IPR_AIAbilityTarget` interface definition
- [[AI System Architecture]] — full system map
- [[Consciousness State Machine]] — state transitions in detail
- [[Disguise Steal]], [[Mind Copy]] — the two abilities this architecture serves

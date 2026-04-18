---
type: decision
title: AI System Architecture
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - technical-architecture
  - adr
  - persival
  - ai
  - sprint
status: implemented
priority: 1
related:
  - '[[APR_BaseAI]]'
  - '[[APR_AIController]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[UPR_EnemyConfig]]'
  - '[[IPR_AIAbilityTarget]]'
  - '[[Consciousness State Machine]]'
  - '[[Ability System Interface]]'
  - '[[game-design/technical-architecture/_index]]'
---

# AI System Architecture

**Project:** Persival
**Status:** Implemented — AI Sprint
**Scope:** All NPC behavior, player-ability interaction, global suspicion

---

## Class Hierarchy

```
ATurnInPlaceCharacter (Plugin: TurnInPlace-release/)
  └── APR_BaseCharacter
        ├── APR_BasePlayer  (+IPR_InputProvider)   ← P1 / P2
        └── APR_BaseAI      (+IPR_AIAbilityTarget)  ← Guards, Officers, Civilians
              controlled by APR_AIController
```

---

## Subsystem Map

```
APR_BaseAI
 ├── UPR_EnemyConfig       (DataAsset — all tunable params)
 ├── UPR_AIMemoryComponent (consciousness state + snapshots)
 ├── UPR_AIPerceptionComponent (sight/hearing, reads config)
 └── RuntimeTags            (FGameplayTagContainer, replicated)

APR_AIController
 ├── Blackboard (BB_AIBase)
 ├── BehaviorTree (BT_AIBase)
 └── P2 entry points: ApplyPasteA(), ApplyPasteB()

UPR_SuspicionSubsystem (GameInstanceSubsystem)
 └── Global alert accumulator — persists across subsections
```

---

## Behavior Tree Priority (`BT_AIBase`)

| Priority | Condition | Behavior |
|----------|-----------|----------|
| 1 (highest) | `ConsciousnessState != Normal` | Run `BT_ControlledByPlayer` subtree |
| 2 | `AlertLevel >= 1.0` | `BTTask_PR_TriggerFailure` → GameMode failure |
| 3 | `AlertLevel >= 0.5` | Move to `LastKnownLocation` → wait → search → `BTTask_PR_ClearAlert` |
| 4 (lowest) | `AlertLevel < 0.5` | Follow `APR_PatrolPath` or wait at point |

---

## Ability Interaction Flow

### P1 FaceSteal — Identity Hijack
```
Ability activates on target NPC
  → IPR_AIAbilityTarget::CanBeTargetedByAbility()      [gate]
  → APR_BaseAI::GetRuntimeTags()                        [copy tags to player]
  → UPR_AIMemoryComponent::SetConsciousnessState(Frozen)
  → BTTask_PR_HandleFrozen holds NPC indefinitely
  → P1 within FaceStealRange: tags active on player
  → AI perception reads player tags → "friendly guard"
  → Deactivate: RemoveGameplayTag + SetConsciousnessState(Normal)
```

### P2 MindCopy — Copy
```
Ability activates on source NPC
  → IPR_AIAbilityTarget::GetMemoryComponent()
  → UPR_AIMemoryComponent::TakeSnapshot(Blackboard)
  → FAIConsciousnessSnapshot stored (server-only, one per P2)
```

### P2 Paste A — Restore
```
Ability activates on target NPC
  → APR_AIController::ApplyPasteA()
  → BB: OverwrittenEndTime = Now + DisorientDuration, MindReplaceEndTime = 0
  → ConsciousnessState → Overwritten
  → BTTask_PR_HandleOverwritten counts down
  → MindReplaceEndTime == 0 detected → RestoreFromSnapshot() → Normal
```

### P2 Paste B — Replacement
```
Ability activates on target NPC
  → APR_AIController::ApplyPasteB()
  → UPR_AIMemoryComponent::TakeOwnStateSnapshot()       [save target state]
  → BB: OverwrittenEndTime, MindReplaceEndTime set
  → ConsciousnessState → Overwritten → Replaced
  → BTService_PR_MindReplaceExpiry polls MindReplaceEndTime
  → Expiry: RestoreFromOwnStateSnapshot() + +1 global suspicion → Normal
```

---

## Core Design Principles

### 1. Data-Driven
All enemy parameters live in `UPR_EnemyConfig` DataAssets — nothing hardcoded in C++. New enemy archetype = new DataAsset + thin Blueprint subclass.

### 2. State-Driven
`ConsciousnessState` is the root BT branch condition. Ability effects change state → BT re-evaluates → behavior changes. No imperative "do X" calls from ability code into BT.

### 3. Interface-First
Player abilities interact with AI exclusively via `IPR_AIAbilityTarget`. No concrete casts. Mechanical enemies return `CanBeTargetedByAbility() = false` — immune.

### 4. GameplayTags for Identity
AI perceives friend/foe via tags (`AI.Type.Guard`, `AI.Faction.Mansion`), not mesh or class. P1 disguise = tag injection. Zero perception override hooks needed.

### 5. Blackboard as State Bus
All BT tasks read/write Blackboard. No C++-side state duplication. Snapshots are a full BB capture → restore cycle.

---

## Why Not GAS?

| Criterion | Custom BT+BB+Interface | Unreal GAS |
|-----------|----------------------|------------|
| Team size fit | 3-person team, prototype | Large footprint, steep curve |
| Replication | `ConsciousnessState` replicated; snapshots server-only | Built-in prediction |
| Ability complexity | 2 abilities, BT-driven | Overkill |
| Extensibility | New ability = new BT task + interface method | New ability = GAS boilerplate |
| Future multiplayer | Deferred prediction pass planned | GAS handles it natively |

**Decision:** Custom interface + BT-driven state for prototype. If multiplayer becomes a serious goal after 80% mechanic completion, revisit GAS as an adapter layer.

---

## Replication Notes

Per CLAUDE.md: add replication where it costs little, defer complex prediction.

| What | Status |
|------|--------|
| `RuntimeTags` (`APR_BaseAI`) | Replicated — clients need identity for cosmetics |
| `ConsciousnessState` (`UPR_AIMemoryComponent`) | Replicated — drives client-side cosmetic reactions |
| Snapshots (`Snapshot`, `OwnStateSnapshot`) | Server-only — BT runs server-side |
| `AlertLevel` in Blackboard | Server-authoritative, not replicated directly |

---

## Key Source Files

| File | Role |
|------|------|
| `PR_BaseAI.h/cpp` | NPC pawn, tag system, interface impl |
| `PR_AIController.h/cpp` | Perception, alert, P2 entry points |
| `PR_AIMemoryComponent.h/cpp` | Consciousness state, snapshot API |
| `PR_EnemyConfig.h` | DataAsset — all params |
| `PR_AITypes.h` | `EAIConsciousnessState`, `FAIConsciousnessSnapshot` |
| `PR_AIBlackboardKeys.h` | Centralized BB key names |
| `PR_AIAbilityTarget.h` | Interface — ability interaction boundary |
| `BTTask_PR_HandleFrozen` | Frozen state lock |
| `BTTask_PR_HandleOverwritten` | Paste A/B disorientation + branch |
| `BTService_PR_MindReplaceExpiry` | Paste B expiry + restore |
| `PR_SuspicionSubsystem.h` | Global suspicion GameInstanceSubsystem |

---

## Related
- [[Consciousness State Machine]] — state transition detail
- [[Ability System Interface]] — `IPR_AIAbilityTarget` implementation reference
- [[APR_BaseAI]], [[APR_AIController]], [[UPR_AIMemoryComponent]], [[UPR_EnemyConfig]]
- [[Disguise Steal]], [[Mind Copy]] — mechanic pages

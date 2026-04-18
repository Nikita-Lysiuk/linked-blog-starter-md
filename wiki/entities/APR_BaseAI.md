---
type: class
title: APR_BaseAI
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - class
  - ai
  - cpp
  - persival
module: AI
header: Source/Nocturne/Public/AI/PR_BaseAI.h
related:
  - '[[IPR_AIAbilityTarget]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[UPR_EnemyConfig]]'
  - '[[APR_AIController]]'
  - '[[Consciousness State Machine]]'
  - '[[Disguise Steal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
---

# APR_BaseAI

**Header:** `Source/Nocturne/Public/AI/PR_BaseAI.h`
**Module:** AI
**Inheritance:** `APR_BaseCharacter` → `ATurnInPlaceCharacter` (plugin) | implements [[IPR_AIAbilityTarget]]

Base pawn for all NPCs in Nocturne — Guards, Officers, Civilians. Manages identity tags, consciousness state delegation, patrol configuration, and the ability-targeting interface.

---

## Class Declaration

```cpp
UCLASS(Abstract, Blueprintable)
class APR_BaseAI : public APR_BaseCharacter, public IPR_AIAbilityTarget
```

---

## Key Properties

| Property | Type | Line | Notes |
|----------|------|------|-------|
| `EnemyConfig` | `TObjectPtr<UPR_EnemyConfig>` | 145 | Data-driven config — all stats, perception, ability flags |
| `PatrolPath` | `TObjectPtr<APR_PatrolPath>` | 153 | Assigned patrol path actor |
| `StartPatrolIndex` | `int32` | 161 | Starting waypoint (offset for shared paths) |
| `AllowedPatrolIndices` | `TArray<int32>` | 181 | Optional sub-list of reachable waypoints |
| `MemoryComponent` | `TObjectPtr<UPR_AIMemoryComponent>` | 184 | Consciousness + snapshot management |
| `RuntimeTags` | `FGameplayTagContainer` | 196 | **Replicated.** Identity tags read by AI perception. Mutated by P1 FaceSteal. |

---

## IPR_AIAbilityTarget Implementation

| Method | Line | Implementation |
|--------|------|----------------|
| `CanBeTargetedByAbility_Implementation()` | 85 | `return EnemyConfig->bCanBeTargetedByAbility` |
| `GetMemoryComponent_Implementation()` | 90 | `return MemoryComponent` |
| `GetCurrentConsciousnessState_Implementation()` | 95 | Delegates to `MemoryComponent->GetConsciousnessState()` |

---

## Identity / Tag Methods

| Method | Line | Notes |
|--------|------|-------|
| `GetRuntimeTags()` | 122 | `const FGameplayTagContainer&` |
| `HasGameplayTag(FGameplayTag)` | 125 | Tag query |
| `AddGameplayTag(FGameplayTag)` | 128 | P1 injects faction tags onto NPC (or vice versa onto player) |
| `RemoveGameplayTag(FGameplayTag)` | 133 | Called on FaceSteal deactivation |
| `InitializeRuntimeTags()` | cpp:129 | Copies `EnemyConfig->DefaultTags` at `BeginPlay` — **server only** |

`RuntimeTags` replication: initialized server-only, `OnRep_RuntimeTags` keeps clients current. AI perception reads tags at query time — not cached — so tag changes take effect immediately.

---

## Blueprint Events

| Method | Line | Trigger |
|--------|------|---------|
| `OnArriveAtPatrolPoint()` | 93 | `BlueprintNativeEvent` — fired on patrol point arrival |
| `OnNoticePlayer()` | 103 | `BlueprintNativeEvent` — investigation start |
| `OnSuspicionModeActivated()` | 115 | `BlueprintNativeEvent` — AlertLevel >= 0.5 threshold |

---

## Identity Tag System (P1 Disguise Foundation)

AI perception identifies friend/foe by querying `RuntimeTags` — **not** by mesh or class type. P1's FaceSteal copies a target NPC's tags onto the player, making every NPC's perception query return "friendly". Key consequences:

- Two NPCs of the same role (same `DefaultTags`) are perceptually indistinguishable.
- Tag changes are replicated and take effect for all perception queries immediately.
- Mechanical enemies have `bCanBeTargetedByAbility = false` — immune to both P1 and P2.

See [[Disguise Steal]] for the full mechanic and [[IPR_AIAbilityTarget]] for the interface contract.

---

## Relationships

| Relationship | Target |
|-------------|--------|
| Config | [[UPR_EnemyConfig]] — all data-driven parameters |
| Memory | [[UPR_AIMemoryComponent]] — consciousness state + P2 snapshots |
| Controller | [[APR_AIController]] — perception, alert level, ability entry points |
| Interface | [[IPR_AIAbilityTarget]] — P1/P2 interaction boundary |
| P1 mechanic | [[Disguise Steal]] — reads/writes `RuntimeTags` + sets Frozen |
| P2 mechanic | [[Mind Copy]] — reads/writes snapshots via MemoryComponent |
| Architecture | [[AI System Architecture]] |

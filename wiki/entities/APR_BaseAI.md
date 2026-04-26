---
type: class
title: APR_BaseAI
created: '2026-04-18'
updated: '2026-04-27'
tags:
  - class
  - ai
  - cpp
  - persival
status: implemented
module: AI
header: Source/Nocturne/Public/AI/PR_BaseAI.h
related:
  - '[[IPR_AIAbilityTarget]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[UPR_EnemyConfig]]'
  - '[[APR_AIController]]'
  - '[[Consciousness State Machine]]'
  - '[[FaceSteal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
---

# APR_BaseAI

**Header:** `Source/Nocturne/Public/AI/PR_BaseAI.h`
**Module:** AI
**Inheritance:** `APR_BaseCharacter` → `ATurnInPlaceCharacter` (plugin) | implements [[IPR_AIAbilityTarget]]

Base pawn for all NPCs in Nocturne — Guards, Officers, Civilians. Manages consciousness state delegation, patrol configuration, and the ability-targeting interface. Identity tags live on the parent `APR_BaseCharacter`.

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

---

## IPR_AIAbilityTarget Implementation

| Method | Implementation |
|--------|----------------|
| `CanBeTargetedByAbility_Implementation()` | `return EnemyConfig->bCanBeTargetedByAbility` |
| `GetMemoryComponent_Implementation()` | `return MemoryComponent` |
| `BeginDisorientation_Implementation()` | Sets `bIsDisoriented = true` on Blackboard. BP override plays stagger animation. |
| `GetCurrentConsciousnessState_Implementation()` | Delegates to `MemoryComponent->GetConsciousnessState()` |

---

## Identity / Tag System

`RuntimeTags` and all tag CRUD methods (`GetRuntimeTags`, `HasGameplayTag`, `AddGameplayTag`, `RemoveGameplayTag`) are inherited from **`APR_BaseCharacter`** — they were moved there so `APR_BasePlayer` can also hold stolen tags for P1's FaceSteal mechanic.

`APR_BaseAI` retains only `InitializeRuntimeTags()` (private, called at `BeginPlay`) which copies `EnemyConfig->DefaultTags` into the inherited `RuntimeTags` container. This is AI-specific logic — players don't have an EnemyConfig.

---

## Blueprint Events

| Method | Line | Trigger |
|--------|------|---------|
| `OnArriveAtPatrolPoint()` | 93 | `BlueprintNativeEvent` — patrol point arrival |
| `OnNoticePlayer()` | 103 | `BlueprintNativeEvent` — investigation start |
| `OnSuspicionModeActivated()` | 115 | `BlueprintNativeEvent` — AlertLevel >= 0.5 |

---

## Relationships

| Relationship | Target |
|-------------|--------|
| Config | [[UPR_EnemyConfig]] — all data-driven parameters |
| Memory | [[UPR_AIMemoryComponent]] — consciousness state + P2 snapshots |
| Controller | [[APR_AIController]] — perception, alert level, snapshot restore |
| Interface | [[IPR_AIAbilityTarget]] — P1/P2 interaction boundary |
| P1 mechanic | [[FaceSteal]] — copies RuntimeTags (from APR_BaseCharacter) + sets Frozen |
| P2 mechanic | [[Mind Copy]] — reads/writes snapshots via MemoryComponent |
| Architecture | [[AI System Architecture]] |

---
type: class
title: UPR_AIMemoryComponent
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - class
  - component
  - ai
  - cpp
  - persival
module: AI/Components
header: Source/Nocturne/Public/AI/Components/PR_AIMemoryComponent.h
related:
  - '[[APR_BaseAI]]'
  - '[[APR_AIController]]'
  - '[[IPR_AIAbilityTarget]]'
  - '[[Mind Copy]]'
  - '[[Consciousness State Machine]]'
  - '[[AI System Architecture]]'
---

# UPR_AIMemoryComponent

**Header:** `Source/Nocturne/Public/AI/Components/PR_AIMemoryComponent.h`
**Module:** AI/Components
**Base:** `UActorComponent`
**Owner:** [[APR_BaseAI]]

Manages the AI's "Mind" — consciousness state transitions and the snapshot system that powers P2's Copy/Paste abilities. Implements the **Memento pattern**: captures full Blackboard state into a snapshot struct, restores it on demand.

---

## Class Declaration

```cpp
UCLASS(ClassGroup=(AI), meta=(BlueprintSpawnableComponent))
class UPR_AIMemoryComponent : public UActorComponent
```

---

## State Management

| Property | Type | Line | Notes |
|----------|------|------|-------|
| `ConsciousnessState` | `EAIConsciousnessState` | 110 | **Replicated.** Root BT branch condition. |

### EAIConsciousnessState values (`PR_AITypes.h:17–24`)

| State | Meaning |
|-------|---------|
| `Normal` | Default — patrol / alert / investigate |
| `Frozen` | P1 FaceSteal active — NPC locked in place |
| `Overwritten` | P2 Paste A/B disorientation freeze (transitional) |
| `Replaced` | P2 Paste B applied — NPC runs donor's patrol/alert |

### State API

| Method | Line | Notes |
|--------|------|-------|
| `SetConsciousnessState(State)` | 49 | **Server-only.** Broadcasts `OnConsciousnessStateChanged` |
| `GetConsciousnessState()` | 52 | Returns current state |
| `IsConsciousnessNormal()` | 55 | Convenience: `State == Normal` |

### Delegate

```cpp
// Broadcasts on every state change
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnConsciousnessStateChanged,
    EAIConsciousnessState, OldState,
    EAIConsciousnessState, NewState
);
UPROPERTY(BlueprintAssignable) FOnConsciousnessStateChanged OnConsciousnessStateChanged;
// PR_AIMemoryComponent.h:100
```

---

## Snapshot System (P2 Copy/Paste)

Two snapshots are managed, both **server-only** (not replicated):

| Property | Type | Line | Purpose |
|----------|------|------|---------|
| `Snapshot` | `FAIConsciousnessSnapshot` | 116 | **Donor snapshot** — captured by P2 Copy |
| `OwnStateSnapshot` | `FAIConsciousnessSnapshot` | 117 | **Recipient pre-paste state** — saved by Paste B for restore on expiry |

### Donor Snapshot API (P2 Copy → Paste A/B)

| Method | Line | Notes |
|--------|------|-------|
| `TakeSnapshot(UBlackboardComponent*)` | 67 | Captures full BB state into `Snapshot` |
| `GetSnapshot()` | 71 | Returns `Snapshot` — check `bIsValid` first |
| `ClearSnapshot()` | 75 | Consumes snapshot (`bIsValid = false`) |

### Own-State Snapshot API (Paste B restore on expiry)

| Method | Line | Notes |
|--------|------|-------|
| `TakeOwnStateSnapshot(UBlackboardComponent*)` | 86 | Saves recipient's current BB state before Paste B |
| `GetOwnStateSnapshot()` | 90 | Returns `OwnStateSnapshot` |
| `ClearOwnStateSnapshot()` | 94 | Consumes own-state snapshot |

---

## FAIConsciousnessSnapshot (`PR_AITypes.h:76–118`)

```cpp
USTRUCT(BlueprintType)
struct FAIConsciousnessSnapshot
{
    float AlertLevel;
    FVector LastKnownLocation;
    TObjectPtr<AActor> TargetActor;
    EAILastStimulusType LastStimulusType;
    int32 PatrolIndex;
    bool bPatrolReversing;
    EPatrolMode PatrolMode;
    bool bIsValid;   // one-shot flag — ClearSnapshot() sets to false
};
```

---

## Key Design Points

- `SetConsciousnessState` is server-authoritative. `ConsciousnessState` replicates to clients so they can drive cosmetic reactions without extra RPCs.
- Snapshots never replicate — only the server runs BT logic that consumes them.
- `bIsValid` enforces one-shot consumption: `GetSnapshot()` returns an invalid struct if already consumed.
- One snapshot per NPC. New `TakeSnapshot()` call overwrites previous.

---

## Relationships

| Relationship | Target |
|-------------|--------|
| Owner | [[APR_BaseAI]] |
| Exposed via | [[IPR_AIAbilityTarget]]`::GetMemoryComponent()` |
| Used by controller | [[APR_AIController]] — `ApplyPasteA`, `ApplyPasteB`, `RestoreFromSnapshot` |
| P2 mechanic | [[Mind Copy]] — full Copy/Paste flow |
| State machine | [[Consciousness State Machine]] |

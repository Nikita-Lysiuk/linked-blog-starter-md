---
type: concept
title: Consciousness State Machine
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - technical-architecture
  - ai
  - persival
  - state-machine
status: implemented
related:
  - '[[APR_BaseAI]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[APR_AIController]]'
  - '[[Disguise Steal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
  - '[[game-design/technical-architecture/_index]]'
---

# Consciousness State Machine

**Project:** Persival
**Source:** `PR_AITypes.h:17–24` (`EAIConsciousnessState`)
**Owner:** [[UPR_AIMemoryComponent]] — `SetConsciousnessState()` is the only write path

The root branch condition of `BT_AIBase`. When `ConsciousnessState != Normal`, the BT runs the `BT_ControlledByPlayer` subtree instead of the standard patrol/alert/investigate logic.

---

## States

```cpp
UENUM(BlueprintType)
enum class EAIConsciousnessState : uint8
{
    Normal,      // Default — patrol / alert / investigate
    Frozen,      // P1 FaceSteal active — NPC locked in place
    Overwritten, // P2 Paste transitional — brief disorientation freeze
    Replaced     // P2 Paste B — NPC runs donor's patrol/alert state
};
// PR_AITypes.h:17–24
```

---

## State Transition Diagram

```
                    ┌─────────────────────────────┐
                    │                             ▼
             P1 FaceSteal            ┌──────────────────┐
Normal ──────────────────────────► Frozen               │
  ▲                                  │ P1 deactivates   │
  │ ◄────────────────────────────────┘                  │
  │                                                     │
  │         P2 Paste A/B                               │
  │ ──────────────────────────────► Overwritten         │
  │                                  │                  │
  │  MindReplaceEndTime == 0         │ DisorientDuration │
  │ ◄────────────────────────────────┘  expires         │
  │         (Paste A restore)                           │
  │                                  │ MindReplaceEndTime > 0
  │                                  ▼                  │
  │                              Replaced               │
  │  MindReplaceDuration expires     │                  │
  │ ◄────────────────────────────────┘                  │
  │         (+1 global suspicion)                       │
  │                                                     │
  └─────────────────────────────────────────────────────┘
```

---

## State Details

### Normal
- Default state for all NPC archetypes at spawn.
- BT runs standard priority selector: Patrol → Suspicion → High Alert.
- `AlertLevel` decayed by `BTService_PR_DecayAlert`, updated by `BTService_PR_UpdateAlertFromSight`.

### Frozen (`P1 FaceSteal`)
- Set by: P1 ability → `UPR_AIMemoryComponent::SetConsciousnessState(Frozen)`
- BT task: `BTTask_PR_HandleFrozen` — returns `InProgress` indefinitely
- Exit: P1 deactivates → `SetConsciousnessState(Normal)` externally aborts the task
- NPC position: held at current location; no movement, no perception updates
- NPC state after: resumes BT from root — current `AlertLevel` unchanged

### Overwritten (transitional)
- Set by: `APR_AIController::ApplyPasteA()` or `ApplyPasteB()`
- BT task: `BTTask_PR_HandleOverwritten` — counts down from `OverwrittenEndTime`
- Duration: `DisorientDuration` (configured in `UPR_EnemyConfig`)
- Exit branches:
  - `MindReplaceEndTime == 0` → **Paste A path**: `RestoreFromSnapshot()` → `Normal`
  - `MindReplaceEndTime > Now` → **Paste B path**: apply snapshot → `Replaced`
- Represents brief disorientation / confusion before the consciousness settles

### Replaced (`P2 Paste B`)
- Set by: `BTTask_PR_HandleOverwritten` on Paste B branch
- BT task: standard BT runs but with **donor's Blackboard state** applied
  - NPC walks donor's patrol route, inherits donor's `AlertLevel` and `TargetActor`
- Expiry monitor: `BTService_PR_MindReplaceExpiry` polls `MindReplaceEndTime` each tick
- On expiry:
  1. `APR_AIController::RestoreFromOwnStateSnapshot()` — restores recipient's pre-paste state
  2. `UPR_SuspicionSubsystem` += 1 global suspicion
  3. `SetConsciousnessState(Normal)` — NPC wakes disoriented at wrong location

---

## Write Path

`SetConsciousnessState()` is the **only** way to change state. It is:
- **Server-only** — authoritative state change
- Broadcasts `OnConsciousnessStateChanged(OldState, NewState)` delegate immediately
- State replicates to clients via `ConsciousnessState` `UPROPERTY(Replicated)`

No BT task sets state directly — they all call back through the controller or MemoryComponent.

---

## Blackboard Keys Involved

| Key | Source | Role |
|-----|--------|------|
| `OverwrittenEndTime` | `PR_AIBlackboardKeys.h:94` | World time when disorientation ends |
| `MindReplaceEndTime` | `PR_AIBlackboardKeys.h:100` | World time when Paste B ends; `0` = Paste A |

---

## Key Interaction: P1 + P2 on Same NPC

| Scenario | Result |
|----------|--------|
| P1 freezes NPC (Frozen), then P2 Paste A | `RestoreFromSnapshot()` sets `Normal` → **P1 disguise breaks** |
| P1 freezes NPC (Frozen), then P2 Paste B | `Overwritten` overwrites `Frozen` → P1 disguise breaks, NPC runs replacement route |
| P2 Paste B (Replaced), then P1 tries FaceSteal | P1 can target Replaced NPC (state != Normal, but `CanBeTargetedByAbility` still true) — interaction undefined, needs ADR |

---

## Related
- [[UPR_AIMemoryComponent]] — owns `ConsciousnessState` and write API
- [[APR_AIController]] — initiates Overwritten / provides snapshot restore
- [[Disguise Steal]] — produces Frozen transitions
- [[Mind Copy]] — produces Overwritten / Replaced transitions
- [[AI System Architecture]] — full system context

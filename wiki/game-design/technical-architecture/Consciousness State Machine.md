---
type: concept
title: Consciousness State Machine
created: '2026-04-18'
updated: '2026-04-27'
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
  - '[[FaceSteal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
  - '[[game-design/technical-architecture/_index]]'
---

# Consciousness State Machine

**Project:** Persival
**Source:** `PR_AITypes.h` (`EAIConsciousnessState`)
**Owner:** [[UPR_AIMemoryComponent]] — `SetConsciousnessState()` is the only write path

The root branch condition of `BT_AIBase`. When `ConsciousnessState != Normal`, the BT runs the `BT_ControlledByPlayer` subtree instead of the standard patrol/alert/investigate logic.

---

## States

```cpp
UENUM(BlueprintType)
enum class EAIConsciousnessState : uint8
{
    Normal,   // Default — patrol / alert / investigate
    Frozen,   // P1 FaceSteal active — NPC locked in place
    Replaced  // P2 Paste B — NPC runs donor's patrol/alert state
};
// PR_AITypes.h
```

> [!note] Overwritten removed (2026-04-26)
> `EAIConsciousnessState::Overwritten` was deleted. Transitional disorientation is now handled by the `bIsDisoriented` Blackboard key + `BTTask_PR_Disorient`. No separate consciousness state is needed — the BT reads a bool and the task handles the wake-up window.

---

## State Transition Diagram

```
                P1 FaceSteal (UPR_FaceStealComponent)
Normal ──────────────────────────────────────────► Frozen
  ▲                                                    │
  │  BTTask_PR_Disorient (2s) ◄── BeginDisorientation() on deactivate
  │ ◄──────────────────────────────────────────────────┘

  │         P2 Paste B (UPR_TelepathyComponent — pending PR-91)
  │ ──────────────────────────────────────────────► Replaced
  │  MindReplaceDuration expires                       │
  │ ◄──────────────────────────────────────────────────┘
  │         (+1 global suspicion, RestoreFromOwnStateSnapshot)
```

---

## State Details

### Normal
- Default state at spawn for all archetypes.
- BT runs standard priority selector: Patrol → Suspicion → High Alert.
- `AlertLevel` managed by `BTService_PR_DecayAlert` and `BTService_PR_UpdateAlertFromSight`.

### Frozen (`P1 FaceSteal`)
- **Set by:** `UPR_FaceStealComponent::Activate_Implementation()` → `UPR_AIMemoryComponent::SetConsciousnessState(Frozen)`
- **BT task:** `BTTask_PR_HandleFrozen` — returns `InProgress` indefinitely
- **Exit path:** P1 deactivates → `IPR_AIAbilityTarget::BeginDisorientation()` called → `bIsDisoriented` BB key set → `BTTask_PR_Disorient` holds NPC for `DisorientDuration` (~2s) → `SetConsciousnessState(Normal)`
- **NPC during Frozen:** held at current location, no movement, no perception updates
- **AlertLevel on wake:** 0 — the disorientation window is the player's escape

### Replaced (`P2 Paste B`)
- **Set by:** `UPR_TelepathyComponent` (pending — PR-91) → `UPR_AIMemoryComponent`
- **BT behaviour:** standard BT runs with **donor's Blackboard state** applied — NPC walks donor's patrol, inherits donor's `AlertLevel` and `TargetActor`
- **Expiry monitor:** `BTService_PR_MindReplaceExpiry` polls `MindReplaceEndTime` each tick
- **On expiry:**
  1. `APR_AIController::RestoreFromOwnStateSnapshot()` — restores recipient's pre-paste BB state
  2. `UPR_SuspicionSubsystem` += 1 global suspicion
  3. `SetConsciousnessState(Normal)` — NPC wakes at wrong location

---

## Write Path

`SetConsciousnessState()` is the **only** way to change state:
- **Server-only** — authoritative
- Broadcasts `OnConsciousnessStateChanged(OldState, NewState)` delegate immediately
- Replicates to clients via `ConsciousnessState` `UPROPERTY(Replicated)`

---

## Blackboard Keys Involved

| Key | Role |
|-----|------|
| `bIsDisoriented` | Set by `BeginDisorientation()` — drives `BTTask_PR_Disorient` wake-up |
| `MindReplaceEndTime` | World time when Paste B expires; read by `BTService_PR_MindReplaceExpiry` |

---

## Key Interaction: P1 + P2 on Same NPC

| Scenario | Result |
|----------|--------|
| P1 Frozen NPC, then P2 Paste B | `SetConsciousnessState(Replaced)` overwrites Frozen → P1 disguise breaks. Needs ADR once Telepathy is built (PR-91). |
| P2 Replaced NPC, then P1 FaceSteal | `CanBeTargetedByAbility()` still true → P1 can target. Interaction undefined — needs ADR. |

---

## Related
- [[UPR_AIMemoryComponent]] — owns state and write API
- [[APR_AIController]] — provides snapshot restore for Replaced expiry
- [[FaceSteal]] — produces Frozen transitions
- [[Mind Copy]] — produces Replaced transitions
- [[AI System Architecture]] — full system context

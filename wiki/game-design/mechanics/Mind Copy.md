---
type: mechanic
title: Mind Copy (MindCopy)
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - mechanic
  - sprint
  - persival
  - p2
  - mind-manipulation
status: implementing
priority: 1
loop_tier: core
related:
  - '[[IPR_AIAbilityTarget]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[APR_AIController]]'
  - '[[Consciousness State Machine]]'
  - '[[Disguise Steal]]'
  - '[[game-design/mechanics/_index]]'
sources: []
---

# Mind Copy (MindCopy)

**Project:** Persival — P2 ability
**Loop Tier:** Core
**Status:** `implementing` — BT/BB/MemoryComponent infrastructure complete. Player-facing component TBD.

---

## Design Intent

P2 captures a *snapshot* of an NPC's consciousness at a specific moment — their patrol position, alert level, last known threat, movement state. That snapshot can be pasted back into the same NPC (making them forget) or into a different NPC (making them walk someone else's route for a duration). P2 holds exactly **one snapshot** at a time. New Copy overwrites old.

---

## Two Verbs: Copy and Paste

### Copy
- P2 targets any NPC via `IPR_AIAbilityTarget` → `GetMemoryComponent()`.
- `UPR_AIMemoryComponent::TakeSnapshot()` captures the full Blackboard state into `FAIConsciousnessSnapshot`.
- NPC feels nothing — state is unchanged.
- **One snapshot per P2.** New Copy overwrites previous.

**Fields captured** (`FAIConsciousnessSnapshot`, `PR_AITypes.h:76–118`):**

| Field | Purpose |
|-------|---------|
| `AlertLevel` | Suspicion level at time of copy |
| `LastKnownLocation` | Where target was last seen |
| `TargetActor` | Tracked hostile |
| `LastStimulusType` | Sight / hearing |
| `PatrolIndex` | Active waypoint |
| `bPatrolReversing` | Patrol direction |
| `PatrolMode` | Loop / pingpong / wait |
| `bIsValid` | One-shot flag — consumed on Paste |

---

### Paste A — Restore
Rewinds the target NPC to the copied moment. Can be applied to **any** NPC (same or different from Copy source).

**Flow:**
1. `APR_AIController::ApplyPasteA()` called — `PR_AIController.h:97`.
2. `OverwrittenEndTime = Now + DisorientDuration`, `MindReplaceEndTime = 0` (Paste A marker).
3. `ConsciousnessState` → **`Overwritten`** (disorientation freeze).
4. `BTTask_PR_HandleOverwritten` counts down `DisorientDuration`.
5. Detects `MindReplaceEndTime == 0` → Paste A branch.
6. Snapshot written back to Blackboard via `RestoreFromSnapshot()`.
7. `ConsciousnessState` → **`Normal`** — NPC resumes with copied state.

**Effect:** NPC briefly disoriented, then continues at the copied alert/patrol/target. Applied to an alerted NPC → they forget the sighting.

---

### Paste B — Replacement
Sends the target NPC on the source NPC's route for `MindReplaceDuration` seconds.

**Flow:**
1. `APR_AIController::ApplyPasteB()` called — `PR_AIController.h:100`.
2. Target's current BB state saved → `OwnStateSnapshot` (for restoration on expiry).
3. `OverwrittenEndTime = Now + DisorientDuration`, `MindReplaceEndTime = Now + DisorientDuration + MindReplaceDuration`.
4. `ConsciousnessState` → **`Overwritten`**.
5. `BTTask_PR_HandleOverwritten` counts down `DisorientDuration`.
6. Detects `MindReplaceEndTime > Now` → Paste B branch.
7. Snapshot (source's state) applied to target's Blackboard.
8. `ConsciousnessState` → **`Replaced`** — target walks source's patrol/alert route.
9. On expiry: `BTService_PR_MindReplaceExpiry` restores target from `OwnStateSnapshot`, adds **+1 global suspicion**, sets state → **`Normal`**.

**Effect:** Target NPC covers source NPC's route for the duration — opens a patrol window. On expiry, target wakes disoriented at the wrong location. Costs +1 global suspicion.

---

## State Transition Summary

```
Copy:    No NPC change                       Snapshot held by P2
Paste A: Normal → Overwritten → Normal       Brief freeze, then rewound BB state
Paste B: Normal → Overwritten → Replaced → Normal
         Freeze → source route → snap-back (+1 suspicion)
```

---

## Cooperative Use with P1

| Scenario | Result |
|----------|--------|
| P2 Paste A on NPC that spotted P1 | NPC forgets — alert reset before reaching 1.0 |
| P2 Paste B on third NPC while P1 wears a guard's identity | Guard covers another route — cooperative path clear |
| P2 Paste A on P1's currently Frozen NPC | Restores `Normal` → **breaks P1's disguise** (unresolved edge case) |

---

## Technical Implementation

| Concern | Source |
|---------|--------|
| Copy entry point | `UPR_AIMemoryComponent::TakeSnapshot()` — `PR_AIMemoryComponent.h:67` |
| Paste A entry point | `APR_AIController::ApplyPasteA()` — `PR_AIController.cpp:162` |
| Paste B entry point | `APR_AIController::ApplyPasteB()` — `PR_AIController.cpp:186` |
| Own-state save | `UPR_AIMemoryComponent::TakeOwnStateSnapshot()` — `PR_AIMemoryComponent.h:86` |
| BT disorientation handler | `BTTask_PR_HandleOverwritten.cpp:20–94` |
| Paste B expiry service | `BTService_PR_MindReplaceExpiry.cpp:23–73` |
| BB timing keys | `OverwrittenEndTime` (line 94), `MindReplaceEndTime` (line 100) — `PR_AIBlackboardKeys.h` |

**Snapshots are server-only** — not replicated. Clients don't need consciousness details.

---

## Acceptance Criteria

- [x] `UPR_AIMemoryComponent` snapshot API (`TakeSnapshot`, `TakeOwnStateSnapshot`, `ClearSnapshot`)
- [x] `APR_AIController::ApplyPasteA` and `ApplyPasteB`
- [x] `BTTask_PR_HandleOverwritten` branching logic (Paste A vs B detection)
- [x] `BTService_PR_MindReplaceExpiry` + restore + global suspicion increment
- [ ] P2 player-facing Copy/Paste component written
- [ ] Snapshot HUD indicator (held snapshot + remaining MindReplaceDuration)
- [ ] P1+P2 edge case (Paste A on Frozen NPC) defined and tested
- [ ] Global suspicion increment verified in `PR_SuspicionSubsystem`

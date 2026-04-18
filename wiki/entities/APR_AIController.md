---
type: class
title: APR_AIController
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - class
  - ai
  - cpp
  - persival
module: AI
header: Source/Nocturne/Public/AI/PR_AIController.h
related:
  - '[[APR_BaseAI]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[Mind Copy]]'
  - '[[Consciousness State Machine]]'
  - '[[AI System Architecture]]'
---

# APR_AIController

**Header:** `Source/Nocturne/Public/AI/PR_AIController.h`
**Module:** AI
**Base:** `AAIController`

Owns perception, Blackboard management, and alert tracking. Also serves as the **entry point for P2 abilities** — `ApplyPasteA` and `ApplyPasteB` are called directly on the controller.

---

## Responsibilities

1. **Perception** — wraps `UAIPerceptionComponent`; fetches sight/hearing config from the DataAsset
2. **Alert tracking** — updates `AlertLevel` in Blackboard from sight stimuli
3. **Blackboard seeding** — reads `FAIBlackboardSeedData` from `APR_BaseAI::GetBlackboardSeedData()` at possession
4. **P2 ability entry points** — `ApplyPasteA()` / `ApplyPasteB()` / `RestoreFromSnapshot()` / `RestoreFromOwnStateSnapshot()`

---

## P2 Ability Methods

### ApplyPasteA — Restore (`PR_AIController.h:97`, impl `cpp:162–184`)

```cpp
// Rewinds NPC to captured consciousness snapshot
void ApplyPasteA();
```

**Side effects written to Blackboard:**
- `OverwrittenEndTime = Now + DisorientDuration`
- `MindReplaceEndTime = 0` ← Paste A marker
- `ConsciousnessState` → `Overwritten`

After `DisorientDuration`: `BTTask_PR_HandleOverwritten` detects `MindReplaceEndTime == 0` → `RestoreFromSnapshot()` → `Normal`.

---

### ApplyPasteB — Replacement (`PR_AIController.h:100`, impl `cpp:186–210`)

```cpp
// Sends NPC on source NPC's route for MindReplaceDuration
void ApplyPasteB();
```

**Side effects written to Blackboard:**
- Calls `UPR_AIMemoryComponent::TakeOwnStateSnapshot()` — saves recipient's current state
- `OverwrittenEndTime = Now + DisorientDuration`
- `MindReplaceEndTime = Now + DisorientDuration + MindReplaceDuration`
- `ConsciousnessState` → `Overwritten`

After `DisorientDuration`: task applies donor snapshot → `Replaced`. After `MindReplaceDuration`: `BTService_PR_MindReplaceExpiry` restores `OwnStateSnapshot` → `Normal` + `+1` global suspicion.

---

### Snapshot Restore Methods (`cpp:226–292`)

| Method | Called by | Action |
|--------|-----------|--------|
| `RestoreFromSnapshot()` | `BTTask_PR_HandleOverwritten` (Paste A branch) | Writes donor BB state, clears snapshot |
| `RestoreFromOwnStateSnapshot()` | `BTService_PR_MindReplaceExpiry` (Paste B expiry) | Restores recipient's pre-paste state |

---

## Blackboard Keys Used (`PR_AIBlackboardKeys.h`)

| Key | Line | Purpose |
|-----|------|---------|
| `AlertLevel` | — | Current suspicion (0.0–1.0) |
| `LastKnownLocation` | — | Last hostile sighting |
| `OverwrittenEndTime` | 94 | World time when disorientation ends |
| `MindReplaceEndTime` | 100 | World time when Paste B effect ends; `0` = Paste A |

---

## BT Task / Service Collaborators

| Class | Role |
|-------|------|
| `BTTask_PR_HandleFrozen` | Holds `Frozen` state until external `SetConsciousnessState(Normal)` |
| `BTTask_PR_HandleOverwritten` | Counts down DisorientDuration; branches Paste A vs Paste B |
| `BTService_PR_MindReplaceExpiry` | Polls `MindReplaceEndTime` during `Replaced`; triggers restore on expiry |
| `BTService_PR_DecayAlert` | Decays `AlertLevel` over time when no stimuli |
| `BTService_PR_UpdateAlertFromSight` | Updates `AlertLevel` from sight perception each tick |

---

## Relationships

| Relationship | Target |
|-------------|--------|
| Possessed pawn | [[APR_BaseAI]] |
| Memory | [[UPR_AIMemoryComponent]] — via pawn |
| P2 mechanic | [[Mind Copy]] — `ApplyPasteA/B` are the entry points |
| State machine | [[Consciousness State Machine]] |
| Architecture | [[AI System Architecture]] |

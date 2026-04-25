---
type: mechanic
title: Telepathy — P2 Ability
created: '2026-04-25'
updated: '2026-04-25'
tags:
  - mechanic
  - p2
  - telepathy
  - ability
status: active
owner: P2
---

# Telepathy — P2 Ability

> P2 steals a guard's entire mental state — their alertness, their patrol, their target — and injects it into another guard. Or erases it entirely. Time stops for the victim during integration.

---

## Design Intent

Telepathy is **strategic control over the board**. P1 reacts to the world. P2 reshapes it. Copy is free information; Paste is commitment. The danger is that every Paste creates a freeze window during which the target is visible evidence. Timing Pastes with P1's movement is the core cooperative challenge.

---

## Three Sub-abilities

### Copy
- **What:** Captures target NPC's full Blackboard state into a snapshot
- **How:** P2 aims at target within range, activates Copy
- **NPC reaction:** None. Instant, invisible, undetected
- **Result:** Snapshot stored on the target NPC's `UPR_AIMemoryComponent`
- **Persistence:** Snapshot survives until overwritten by another Copy or consumed by Paste
- **Immunity:** `CanBeTargetedByAbility = false` → Mechs immune

> Copy on its own has zero gameplay effect. It only matters when followed by Paste A or B.

---

### Paste A — Memory Restore
- **What:** Injects a copied snapshot back into the **same** (or a different) NPC, restoring them to their earlier state
- **How:** P2 aims at target, activates Paste A (requires valid snapshot)
- **NPC reaction:**
  1. `ConsciousnessState` → `Overwritten` (`DisorientDuration` freeze, ~1–2 sec)
  2. Player must hide during this window
  3. After freeze: NPC's BB fully restored from snapshot (AlertLevel, patrol index, location)
  4. `ConsciousnessState` → `Normal` — NPC resumes as if nothing happened

**Primary use case:** Panic button. Guard spotted P1 and AlertLevel is rising. P2 Pastes A back to their pre-sighting state. Alert resets. Guard resumes patrol. But P2 must have already Copied that guard — enforces preparation.

**Secondary use case:** Reset a guard to their patrol state after P1 has passed (cleanup).

---

### Paste B — Mind Replace
- **What:** Injects a **different** NPC's consciousness into the target. Target acts exactly as the donor for `MindReplaceDuration` seconds.
- **How:** P2 aims at target, activates Paste B (requires valid snapshot from a *different* NPC)
- **NPC reaction:**
  1. `ConsciousnessState` → `Overwritten` (`DisorientDuration` freeze, ~1–2 sec)
  2. After freeze: target's BB **fully replaced** with donor's state
     - Same `AlertLevel` as donor (if donor was at 0.5, target immediately searches)
     - Same `LastKnownLocation`, same patrol path, same patrol index
     - Same `TargetActor` if donor had one
  3. `ConsciousnessState` → `Replaced` — target runs **normal BT** with donor's data
  4. After `MindReplaceDuration`: target snaps back to their own pre-paste state
     - +1 global suspicion added (NPC is confused, disoriented, somewhere unexpected)
     - `ConsciousnessState` → `Normal`

**Primary use case:** Clear a path. Guard B is blocking P1's route. P2 copies Guard A (who patrols away from P1's path), then Pastes A's consciousness into Guard B. Guard B walks Guard A's route, opening the path. Times out after `MindReplaceDuration`.

---

## Ability Synergy (the preparation loop)

```
P2 Copies Guard A (the one P1 will steal OR the one blocking the path)
  → P1 activates FaceSteal on Guard A (rope appears)
    → IF things go wrong: P2 has Paste A ready as a panic button
    → IF path is blocked: P2 uses the Copy on Guard B instead (Paste B to redirect)
```

**The key constraint:** P2 can only hold one snapshot at a time per NPC. Copying Guard A means the old Guard A snapshot is overwritten. This forces decisions — prepare panic or prepare path, not both simultaneously.

---

## Shared Freeze Window (the cooperative moment)

Both Paste A and Paste B begin with `DisorientDuration`. During this window:
- NPC is visually frozen (standing still, glassy-eyed — BP animation)
- NPC is **aware of their surroundings but cannot act**
- P1 must move out of the NPC's sight line
- P2 must ensure no other guard is watching the freeze happen

This ~1–2 second window is where co-op coordination is most intense.

---

## Immunity Interaction (Act 2–3)

When target NPC has high `SuspicionLevel`:
- Snapshot can still be **taken** (Copy is always free)
- Paste fails to apply: NPC resists, wakes up after `DisorientDuration` with `AlertLevel 0.5`
- No UI indicating the paste was resisted (player discovers this through failure — teaches the immunity mechanic)

---

## Balance Parameters

| Parameter | Default | Notes |
|-----------|---------|-------|
| `DisorientDuration` | 1.5 s | Freeze at start of any Paste |
| `MindReplaceDuration` | 30 s | Duration of Paste B effect |
| P2 ability range | TBD | Line of sight required |

---

## Related
- [[FaceSteal]] — P1's ability; Telepathy prepares and recovers from FaceSteal
- [[AI_Social_Behavior]] — Paste B's disorientation wake-up adds to social suspicion
- [[GDD_Nocturne]] — full game context
- **Code:** `UPR_TelepathyComponent` — PR-89, PR-90, PR-91
- **Code:** `APR_AIController::ApplyPasteA()` / `ApplyPasteB()` — already implemented

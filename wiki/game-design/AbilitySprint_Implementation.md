---
type: plan
title: Ability Sprint — Implementation Plan
created: '2026-04-25'
updated: '2026-04-25'
tags:
  - plan
  - sprint
  - abilities
  - implementation
status: active
sprint: Ability Sprint
---

# Ability Sprint — Implementation Plan

> You write the code. This document tells you exactly what to write and why. No spaghetti.

---

## Priority Order

```
Cleanup → Refactor → PR-87 (FaceSteal full) → PR-89/90/91 (Telepathy full) → AI polish
```

### Pre-PR-87 Cleanup (do first, unblocks everything)
1. Delete `Overwritten` from `EAIConsciousnessState`
2. Delete `BTTask_PR_HandleOverwritten` + `OverwrittenEndTime` BB key
3. Delete `ApplyPasteA` / `ApplyPasteB` from `APR_AIController`
4. Refactor `RuntimeTags` → `APR_BaseCharacter` (move from `APR_BaseAI`)
5. Add `BeginDisorientation()` + `TriggerDisorientation()` to `IPR_AIAbilityTarget`
6. Wire `BTTask_PR_Disorient` + `bIsDisoriented` BB decorator into BT asset

---

## PR-88 — Wire `IPR_AIAbilityTarget` on `APR_BaseAI`

**Why first:** Every ability component calls the AI through this interface. Without it, PR-87/89/90/91 can't be written cleanly.

**Requirements:**
- `APR_BaseAI` implements `IPR_AIAbilityTarget`
- `GetMemoryComponent()` → returns `MemoryComponent` (already exists on `APR_BaseAI`)
- `GetEnemyConfig()` → returns `EnemyConfig` (already exists)
- `CanBeTargetedByAbility()` → returns `EnemyConfig->bCanBeTargetedByAbility`
- `GetAIController()` → returns `Cast<APR_AIController>(GetController())`
- Add `bool bCanBeTargetedByAbility = true` to `UPR_EnemyConfig` if not present (Mechs set to false)
- Verify all existing BT tasks that currently cast to `APR_BaseAI` compile cleanly — they should already use the interface

---

## PR-92 — Apply `AI.State.*` Tags on State Changes

**Why early:** `BTService_PR_CheckColleagues` currently gets `UPR_AIMemoryComponent` via interface to check `ConsciousnessState`. Switching to tag query is cheaper and cleaner.

**Requirements:**
- In `APR_AIController::OnConsciousnessStateChanged`:
  - `Frozen` enter: add `AI.State.Frozen` to pawn's `RuntimeTags`
  - `Frozen` exit: remove `AI.State.Frozen`
  - `Replaced` enter: add `AI.State.Replaced`
  - `Replaced` exit: remove `AI.State.Replaced`
- In `APR_AIController::SetAlertLevel` (or `ModifyAlertLevel`):
  - AlertLevel crosses 0.5 upward: add `AI.State.Alerted`
  - AlertLevel drops below 0.5: remove `AI.State.Alerted`
- `RuntimeTags` on `APR_BaseAI` must be `UPROPERTY(Replicated)`
- Update `BTService_PR_CheckColleagues`: replace component access with `HasTag(AI.State.Frozen)` where possible

---

## PR-87 — `UFaceStealComponent` (P1)

**Lives on:** `APR_BasePlayer` (P1 only — set in P1's Blueprint subclass)

**Requirements:**

### Component properties (EditDefaultsOnly)
- `float GraceTimerDuration` — seconds rope flashes before snapping (default 3s)
- `float AbilityRange` — max distance for activation (default = FaceStealRange from target config, but also readable here as override)

### State machine (internal)
Three states: `Inactive`, `Active`, `GraceTimer`

### Activation input (called from `APR_BasePlayer` input)
1. Find closest valid target within `FaceStealRange` (sphere overlap, filter by `CanBeTargetedByAbility`)
2. If no valid target: no-op
3. If target NPC SuspicionLevel >= threshold: trigger immunity snap (see below)
4. Otherwise:
   - Store reference to `StealTarget`
   - Copy target's `SkeletalMesh` → apply to P1's mesh component
   - Copy target's `RuntimeTags` → add to P1's `RuntimeTags` (on `APR_BaseCharacter`)
   - Add `Player.P1.Disguised` tag to P1
   - Call `StealAppearance()` on target via `IPR_AIAbilityTarget` (sets NPC → Frozen)
   - State → `Active`
   - Fire `BP_OnFaceStealActivated(StealTarget)` (BlueprintImplementableEvent for rope UI)

### Tick (0.2s interval while Active)
1. Check distance to `StealTarget`
2. If `StealTarget` invalid (destroyed): force deactivate
3. If distance > `FaceStealRange`:
   - If state == `Active`: state → `GraceTimer`, start countdown
   - Fire `BP_OnRopeWarning(TimeRemaining)` each tick for UI
4. If distance returns within range while in `GraceTimer`: state → `Active`, cancel timer

### Deactivation (grace timer expires OR manual release)
1. Revert P1 mesh to original (store original mesh on activation)
2. Remove stolen `RuntimeTags` from P1
3. Remove `Player.P1.Disguised` tag
4. Revert P1 team ID → 0
5. Call `TriggerDisorientation()` on target via `IPR_AIAbilityTarget` (sets `bIsDisoriented`, `BTTask_PR_Disorient` handles wake-up)
6. Clear `StealTarget` reference
7. State → `Inactive`
8. Fire `BP_OnFaceStealDeactivated()` for UI cleanup

### Immunity snap (attempted steal on immune NPC)
1. Fire `BP_OnImmunitySnap(Target)` — rope appears and breaks (BP handles VFX)
2. Set target `ConsciousnessState` → `Overwritten`
3. After `DisorientDuration`, target wakes at `AlertLevel 0.5` (handled by `BTTask_PR_HandleOverwritten` — already coded, but override the MindReplaceEndTime check to always go Normal with 0.5 alert on immunity)
   - **Actually:** add an `OverwrittenEndTime` BB key + a `bImmunityWake` BB bool that `BTTask_PR_HandleOverwritten` reads to set alert 0.5 instead of 0 on Normal transition

### BlueprintImplementableEvents (UI hooks, no C++ implementation)
- `BP_OnFaceStealActivated(AActor* Target)`
- `BP_OnRopeWarning(float TimeRemaining, float MaxTime)`
- `BP_OnFaceStealDeactivated()`
- `BP_OnImmunitySnap(AActor* Target)`

---

## PR-89 — `UPR_TelepathyComponent` — Copy

**Lives on:** `APR_BasePlayer` (P2 only)

**Requirements:**
- `float CopyRange` (EditDefaultsOnly, default 2000 cm)
- On activation: line trace to target (must have line of sight), filter by `CanBeTargetedByAbility`
- Call `MemComp->TakeSnapshot(BB)` on target via interface
- Fire `BP_OnCopySuccess(Target)` for UI/VFX
- No NPC reaction, no suspicion, instant
- Store reference to `LastCopiedNPC` (for Paste targeting logic)

---

## PR-90 — `UPR_TelepathyComponent` — Paste A

**Requirements:**
- Separate input binding (or same button with context: no snapshot = Copy, snapshot = Paste)
- On activation: aim at target NPC, call `target->GetAIController()->ApplyPasteA()` via interface
- Requires `MemComp->GetSnapshot().bIsValid == true` on target
- Fire `BP_OnPasteStarted(Target)` and register `BP_OnPasteComplete` to fire after `DisorientDuration`
- P2 must have line of sight to target
- No suspicion on success

---

## PR-91 — `UPR_TelepathyComponent` — Paste B

**Requirements:**
- Distinct input (separate button from Paste A — players must be intentional)
- On activation: aim at recipient NPC
- The snapshot must exist on **a different** NPC (donor ≠ recipient)
- Call `donor->GetMemoryComponent()->TakeSnapshot(donorBB)` if not already done
- Call `recipient->GetAIController()->ApplyPasteB()` via interface
- Snapshot must be transferred: copy donor's `Snapshot` to recipient's `Snapshot` before calling `ApplyPasteB`
  - Add `TransferSnapshotFrom(UPR_AIMemoryComponent* Source)` method to `UPR_AIMemoryComponent`
- Fire `BP_OnPasteBStarted(Donor, Recipient)`, `BP_OnPasteBActive(Recipient, TimeRemaining)`, `BP_OnPasteBExpired(Recipient)`

---

## PR-94 — Place LightZone Volumes in `Lvl_Mind_Swap_Test`

**Not code — level work:**
- Place 3–5 `APR_LightZone` actors covering shadow areas
- Set `DarknessMultiplier`: 0.1 (deep shadow), 0.4 (dim), 0.7 (partial)
- Verify in PIE: `UPR_StealthComponent::IsInDarkZone()` changes on player entry/exit
- Verify alert ramp is visibly slower in dark zones vs open areas

---

## PR-93 — AI Perception Tuning Pass

**Do last — needs all abilities working to tune meaningfully.**

Target feel:
- Guard at 500 cm frontal, full light → AlertLevel 1.0 in ~2 seconds
- Guard at 1500 cm peripheral, dark zone → barely notices after 5 seconds
- After losing sight → search 10–15 sec before giving up
- `SuspiciousColleagueTime` = 10 sec feels right for Act 1 (lenient tutorial)

---

## What's Already Done (do not re-implement)

| System | Status |
|--------|--------|
| `APR_AIController::ApplyPasteA/B` | ❌ Deleted — moves to ability components |
| `UPR_AIMemoryComponent::TakeSnapshot/TakeOwnStateSnapshot` | ✅ Done |
| `BTTask_PR_HandleFrozen` | ✅ Done |
| `BTTask_PR_HandleOverwritten` | ❌ Deleted — replaced by `BTTask_PR_Disorient` |
| `BTTask_PR_Disorient` | ✅ Done (built 2026-04-26) |
| `IPR_PlayerAbility` interface | ✅ Done (built 2026-04-26) |
| `BTService_PR_MindReplaceExpiry` | ✅ Done |
| `BTService_PR_CheckColleagues` | ✅ Done |
| `BTService_PR_UpdateAlertFromSight` | ✅ Done |
| `UPR_StealthComponent` on `APR_BasePlayer` | ✅ Done |
| `APR_LightZone` + `ECC_LightDetection` | ✅ Done |
| GameplayTag set in `DefaultGameplayTags.ini` | ✅ Done |
| `UPR_SuspicionSubsystem` | ✅ Done |

---

## Related
- [[FaceSteal]] — full mechanic spec for PR-87
- [[Telepathy]] — full mechanic spec for PR-89/90/91
- [[AI_Social_Behavior]] — context for PR-92
- [[GDD_Nocturne]] — why any of this exists

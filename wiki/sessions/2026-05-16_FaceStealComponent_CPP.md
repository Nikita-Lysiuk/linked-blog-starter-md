---
type: session
title: Session — UPR_FaceStealComponent CPP Complete
date: '2026-05-16'
updated: '2026-05-16'
tags:
  - session
  - facesteal
  - sprint
  - abilities
  - cpp
  - input
status: complete
---

# Session — UPR_FaceStealComponent CPP Complete (2026-05-16)

> Resume point. Read this + [[hot]] before starting next session.

---

## What Was Done This Session

| Item | Status |
|------|--------|
| Full `UPR_FaceStealComponent.cpp` implementation | ✅ Complete |
| All design decisions grilled and locked | ✅ Done |
| `StolenMesh` + `StolenTags` added to header | ✅ Done |
| `BP_OnRopeSnapped` added to header | ✅ Done |
| `BP_OnCooldownExpired` + `BP_OnCooldownUpdated` added to header | ✅ Done |
| Input registration system (GrappleHook pattern) | ✅ Done |
| `OnToggleInput` 3-state wrapper | ✅ Done |
| 3 minor fixes still needed (see below) | ⚠ Pending |

---

## 3 Remaining Fixes Before Compile

1. **Line 375 in CPP** — `if (IA_AbilityToggle)` inside the confirm binding block → change to `if (IA_AbilityConfirm)`
2. **Header categories** — `BP_OnCooldownExpired` and `BP_OnCooldownUpdated` are under `"Face Steal|TargetMode"` → change to `"Face Steal|Cooldown"`
3. **Unused include** — `"AI/PR_EnemyConfig.h"` not used in CPP → remove

---

## Key Design Decisions Locked This Session

### Input Architecture
- Component owns its own `UInputMappingContext` + `UInputAction` properties (mirrors `UPR_GrappleHookComponent` pattern)
- `APR_BasePlayer` stays generic — no ability-specific code needed
- `OnControllerChanged` delegate handles late possession / splitscreen controller swaps
- Two separate handles: `ToggleActionHandle`, `ConfirmActionHandle`

### Toggle Input (E key)
`OnToggleInput()` wrapper does a state switch:
- `Inactive` → `StartTargeting`
- `Targeting` → `StopTargeting`
- `Active` / `GraceTimer` → `Deactivate`

### CanActivate Semantics
`CanActivate_Implementation()` = `PendingTarget.IsValid() && State == Targeting && !bOnCooldown`
`StartTargeting` guards itself inline — does NOT call `CanActivate`.

### Rope Snap vs Natural Deactivation
- `BP_OnRopeSnapped` fires from `OnGraceTimerExpired` BEFORE `Deactivate` — BP plays tear VFX/sound
- `BP_OnFaceStealDeactivated` always fires at end of `Deactivate` — BP null-checks rope before touching it
- No `bWasSnapped` parameter — SOLID, BP owns defensive logic

### StolenMesh / OriginalMesh Split
- `OriginalMesh` = always P1's original mesh. Never changes after Activate.
- `StolenMesh` = NPC's mesh stored at Activate time.
- On leave radius: `SetSkeletalMeshAsset(OriginalMesh)` — P1 looks like themselves
- On re-enter radius: `SetSkeletalMeshAsset(StolenMesh)` — P1 looks like NPC again
- On Deactivate: `SetSkeletalMeshAsset(OriginalMesh)` — always safe regardless of which state

### Immunity = Runtime Check (not Activation Gate)
`UpdateStealRadius` checks `CanBeTargetedByAbility()` every tick while Active.
If false → calls `OnGraceTimerExpired()` immediately (no grace period).
Ability always activates normally — immunity only snaps the rope mid-steal.

### Disorientation Flow
`Deactivate_Implementation` does NOT call `Execute_BeginDisorientation` directly.
Instead:
1. Sets `bIsDisoriented = true` on NPC's Blackboard (via cast to `APR_BaseAI` → controller → BB)
2. Sets `ConsciousnessState = Normal` (unfreezes NPC via memory component)
BT evaluates → sees `bIsDisoriented = true` → runs `BTTask_PR_Disorient` → task calls `Execute_BeginDisorientation` for animation.

### IsActive Covers GraceTimer
`IsActive_Implementation()` returns true for both `Active` AND `GraceTimer` — steal is still in effect during grace window.

---

## Full Method Summary

| Method | Role |
|--------|------|
| `StartTargeting` | Inline guard (`!Inactive \|\| bOnCooldown`), set state, BP hook |
| `StopTargeting` | Guard (`!Targeting`), clear PendingTarget, BP hook |
| `Activate` | `CanActivate` guard → freeze NPC → steal tags+mesh → add tag → commit state → BP |
| `Deactivate` | Restore mesh/tags → set BB key → unfreeze NPC → cleanup → cooldown → BP |
| `CanActivate` | `PendingTarget.IsValid() && Targeting && !bOnCooldown` |
| `IsActive` | `Active \|\| GraceTimer` |
| `IsTargeting` | `Targeting` |
| `UpdatePendingTarget` | Line trace → delta-only BP_OnTargetFound/Lost |
| `UpdateStealRadius` | Immunity check (→ snap) + distance check (→ grace timer) + mesh swap |
| `UpdateRopeWarning` | Re-enter check (→ Active + StolenMesh) or BP_OnRopeWarning per tick |
| `UpdateCooldown` | BP_OnCooldownUpdated per tick while on cooldown |
| `OnGraceTimerExpired` | `BP_OnRopeSnapped` → `Deactivate` |
| `OnCooldownExpired` | `bOnCooldown = false` → `BP_OnCooldownExpired` |
| `OnToggleInput` | 3-state switch for E key |
| `RegisterInputWithController` | Add IMC + bind both actions |
| `UnregisterInputFromController` | Remove IMC + unbind both handles |
| `OnControllerChanged` | Delegate: unregister old PC, register new PC |

---

## Next Session: UE Editor Work

### Order of attack
1. Fix 3 compile issues (see above)
2. Create `BP_FaceStealComponent` Blueprint subclass
3. Create Input Assets: `IMC_FaceSteal`, `IA_AbilityToggle`, `IA_AbilityConfirm`
4. Add component to Player Blueprint + assign input assets
5. **Targeting VFX**: `BP_OnTargetingStarted/Cancelled` → vignette post-process
6. **NPC outline**: `BP_OnTargetFound/Lost` → Custom Depth stencil
7. **Rope Niagara**: `BP_OnFaceStealActivated` → Niagara beam (start=P1, end=NPC)
8. **Rope warning**: `BP_OnRopeWarning(TimeRemaining, MaxTime)` → drive color/width via Niagara User Variables
9. **Rope snap**: `BP_OnRopeSnapped` → Niagara burst + sound
10. **Cooldown HUD**: `BP_OnCooldownUpdated/Expired` → UMG progress bar
11. **Sound cues** for each state transition

### Niagara Rope Key Facts
- Use **Beam** emitter type — designed for start-point to end-point effects
- Expose `BeamStart` and `BeamEnd` as **User-exposed vector parameters**
- Update them each tick from Blueprint (get P1 location, get NPC location)
- Drive color and width via additional Niagara User Variables fed from `BP_OnRopeWarning`

---

## Related
- [[2026-04-27_FaceStealComponent_Design]] — previous session, header finalized
- [[FaceSteal]] — mechanic spec
- [[hot]] — current sprint state
- [[AbilitySprint_Implementation]] — implementation plan

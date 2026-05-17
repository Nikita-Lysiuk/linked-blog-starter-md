---
type: hot-cache
title: Current Sprint Context
updated: '2026-05-16'
tags:
  - hot
  - sprint
---

# Hot Cache — Ability Sprint

## Current Sprint: Ability Sprint (Sprint 4)

**Goal:** Connect P1 and P2 abilities to the AI system built in AI Sprint. Make the stealth loop actually playable.

## Do Next

1. **Fix 3 compile issues in `PR_FaceStealComponent.cpp`**:
   - Line 375: `if (IA_AbilityToggle)` → `if (IA_AbilityConfirm)` (copy-paste bug in confirm binding)
   - Header: `BP_OnCooldownExpired/Updated` category `"Face Steal|TargetMode"` → `"Face Steal|Cooldown"`
   - Remove unused `"AI/PR_EnemyConfig.h"` include
2. **UE Editor — FaceSteal BP + VFX + input assets** (see session for full order)
3. **Add tag check to `BTService_PR_UpdateAlertFromSight.cpp`** — `static const FGameplayTag FaceStealActiveTag` at file scope, `HasGameplayTag` check in `TickNode` before alert delta

## What Was Just Finished

- `UPR_FaceStealComponent.cpp` **fully implemented** — all methods complete
- Input registration system added — follows `UPR_GrappleHookComponent` pattern
- `OnToggleInput` 3-state wrapper (Inactive/Targeting/Active+GraceTimer → E key)
- `StolenMesh` + `OriginalMesh` split — radius swap pattern works correctly
- Disorientation flow: `Deactivate` sets `bIsDisoriented=true` on BB + `ConsciousnessState=Normal`, BT runs `BTTask_PR_Disorient`
- All design decisions grilled and locked (see [[2026-05-16_FaceStealComponent_CPP]])

## Key Design Decisions Locked

- **`CanActivate`** = `PendingTarget.IsValid() && State==Targeting && !bOnCooldown`
- **`StartTargeting`** guards itself inline — does not call `CanActivate`
- **`BP_OnRopeSnapped`** fires from `OnGraceTimerExpired` BEFORE `Deactivate` — BP null-checks rope in `BP_OnFaceStealDeactivated`
- **Immunity** = runtime tick check in `UpdateStealRadius` → immediate snap (no grace period)
- **`IsActive`** covers both `Active` AND `GraceTimer` states
- **Input** = component-owned IMC + actions, `OnControllerChanged` delegate handles late possession
- **Disorientation** = set BB key + unfreeze NPC in `Deactivate`, BT task calls `BeginDisorientation` internally

## `UPR_FaceStealComponent` — Header Fields Reference

**Designer properties:** `TraceDistance` (500cm), `StealRadius` (1500cm), `GraceTimerDuration` (3s), `CooldownDuration` (15s)
**Input properties:** `FaceStealMappingContext`, `IA_AbilityToggle`, `IA_AbilityConfirm`
**Private state:** `PendingTarget`, `StealTarget`, `OriginalMesh`, `StolenMesh`, `StolenTags`, `CurrentState`, `GraceTimerHandle`, `CooldownTimerHandle`, `bOnCooldown`, `ToggleActionHandle`, `ConfirmActionHandle`, `CachedPC`

## Jira Tickets

| Key | Summary | Status |
|-----|---------|--------|
| PR-87 | UPR_FaceStealComponent (P1) | **In Progress — CPP done, editor next** |
| PR-88 | Wire IPR_AIAbilityTarget on APR_BaseAI | To Do |
| PR-89 | P2 Telepathy — Copy | To Do |
| PR-90 | P2 Telepathy — Paste A | To Do |
| PR-91 | P2 Telepathy — Paste B | To Do |
| PR-93 | AI perception tuning | To Do |
| PR-94 | Place LightZone volumes | To Do |

## Full Design Docs
- [[2026-05-16_FaceStealComponent_CPP]] — this session, CPP complete + editor plan
- [[2026-04-27_FaceStealComponent_Design]] — header design session
- [[2026-04-26_FaceSteal_Sprint]] — sprint kickoff
- [[AbilitySprint_Implementation]] — implementation plan
- [[FaceSteal]] — mechanic spec

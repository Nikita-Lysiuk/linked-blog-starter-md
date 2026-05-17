---
type: hot-cache
title: Current Sprint Context
updated: "2026-05-17"
tags:
  - hot
  - sprint
---

# Hot Cache — Ability Sprint

## Current Sprint: Ability Sprint (Sprint 4)

**Goal:** Connect P1 and P2 abilities to the AI system built in AI Sprint. Make the stealth loop actually playable.

## Do Next — UE Editor

1. **Blackboard asset** — add `ConsciousnessState` as **Int** key
2. **Behavior Tree** — add Blackboard Decorator at root: `ConsciousnessState == 1 (Frozen)`, Abort Self — AI stops all tasks when FaceSteal activates
3. **NPC outline** — `BP_OnTargetFound` → enable Custom Depth on NPC mesh · `BP_OnTargetLost` → disable
4. **Targeting vignette** — `BP_OnTargetingStarted` → post-process effect · `BP_OnTargetingCancelled` → remove
5. **Niagara rope** — `BP_OnFaceStealActivated` → spawn Beam emitter, update start/end each tick · `BP_OnRopeWarning` → drive color/width via User Variables · `BP_OnRopeSnapped` → burst + disable

## FaceSteal C++ — COMPLETE

All code done and tested:
- `UPR_FaceStealComponent.cpp` — all 7 interface methods, input system, state machine
- `UPR_AIMemoryComponent` — refactored: BB as BT authority, replicated field as client channel, two snapshots (donor + own-state), `SetConsciousnessState` writes both
- `BTService_PR_UpdateAlertFromSight` — `FaceStealActiveTag` check skips alert delta while P1 wears a face
- Debug instrumentation added throughout FaceStealComponent (all `TODO: DEBUG`)
- **Jira PR-95 → Done**

## Key Design Decisions Locked

- **`CanActivate`** = `PendingTarget.IsValid() && State==Targeting && !bOnCooldown`
- **`StartTargeting`** guards itself inline — does not call `CanActivate`
- **`BP_OnRopeSnapped`** fires from `OnGraceTimerExpired` BEFORE `Deactivate`
- **Immunity** = runtime tick in `UpdateStealRadius` → immediate snap (no grace period)
- **`IsActive`** covers both `Active` AND `GraceTimer`
- **Input** = component-owned IMC + actions, `OnControllerChanged` handles late possession
- **Disorientation** = set BB `bIsDisoriented=true` + unfreeze via MemComp, BT task owns animation
- **`APR_BasePlayer`** has no `AbilityComponent` — each ability component is self-contained
- **ConsciousnessState** — BB is BT authority (Int key), replicated UPROPERTY is client channel, `SetConsciousnessState` writes both atomically

## `UPR_FaceStealComponent` — Header Fields Reference

**Designer:** `TraceDistance` (500cm), `StealRadius` (1500cm), `GraceTimerDuration` (3s), `CooldownDuration` (15s)
**Input:** `FaceStealMappingContext`, `IA_AbilityToggle` (Pressed), `IA_AbilityConfirm` (Pressed)
**Private:** `PendingTarget`, `StealTarget`, `OriginalMesh`, `StolenMesh`, `StolenTags`, `CurrentState`, `GraceTimerHandle`, `CooldownTimerHandle`, `bOnCooldown`, `ToggleActionHandle`, `ConfirmActionHandle`, `CachedPC`

## Jira Tickets

| Key | Summary | Status |
|-----|---------|--------|
| PR-95 | Implementation of UFaceStealComponent | **Done** |
| PR-96 | Wire logic with AI System | In Progress |
| PR-97 | UI for Ability | To Do |
| PR-98 | VFX for Ability | To Do |
| PR-99 | Sounds for Ability | To Do |
| PR-100 | Test ability | To Do |
| PR-94 | Place APR_LightZone volumes | To Do |

## Full Design Docs
- [[2026-05-16_FaceStealComponent_CPP]] — full CPP session + all decisions
- [[2026-04-27_FaceStealComponent_Design]] — header design session
- [[FaceSteal]] — mechanic spec
- [[AbilitySprint_Implementation]] — implementation plan

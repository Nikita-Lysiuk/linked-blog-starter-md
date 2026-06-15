---
type: entity
title: UPR_FaceStealComponent
updated: "2026-06-15"
tags: [entity, component, facesteal, p1, niagara]
status: stable
---

# UPR_FaceStealComponent

`UActorComponent` — implements `IPR_PlayerAbility`. Owns all FaceSteal logic: targeting, state machine, identity swap, grace timer, Niagara rope.

Jira: **PR-95** (Done)

---

## Header Fields

### Designer-Facing

| Field | Type | Default | Notes |
|---|---|---|---|
| TraceDistance | float | 500cm | Targeting raycast reach |
| StealRadius | float | 1500cm | Max distance while Active before snap |
| GraceTimerDuration | float | 3s | Grace period after radius breach |
| CooldownDuration | float | 15s | Cooldown after deactivation |

### Input

| Field | Type |
|---|---|
| FaceStealMappingContext | UInputMappingContext |
| IA_AbilityToggle | UInputAction (Pressed) |
| IA_AbilityConfirm | UInputAction (Pressed) |

### Effects (Niagara)

| Field | Type | UPROPERTY |
|---|---|---|
| RopeEffectTemplate | `TObjectPtr<UNiagaraSystem>` | EditDefaultsOnly, BlueprintReadOnly |
| RopeEffect | `TObjectPtr<UNiagaraComponent>` | EditAnywhere, BlueprintReadWrite |

Assigned in Class Defaults: `RopeEffectTemplate = NS_FaceStealRope`

### Private State

| Field | Purpose |
|---|---|
| PendingTarget | Weak pointer to NPC under targeting reticle |
| StealTarget | Weak pointer to NPC currently stolen from |
| OriginalMesh | Player's original mesh before swap |
| StolenMesh | NPC mesh applied to player |
| StolenTags | NPC GameplayTags applied to player |
| CurrentState | `EFaceStealState` enum |
| GraceTimerHandle | FTimerHandle for grace period |
| CooldownTimerHandle | FTimerHandle for cooldown |
| bOnCooldown | bool |
| ToggleActionHandle / ConfirmActionHandle | Input binding handles |
| CachedPC | Weak pointer to PlayerController |

---

## State Machine

```
Inactive → Targeting → Active → GraceTimer → Inactive (+ Cooldown)
                     ↘ Inactive (manual cancel, no cooldown)
```

- `CanActivate()` = `PendingTarget.IsValid() && State==Targeting && !bOnCooldown`
- `StartTargeting()` guards inline — does not call `CanActivate`
- `IsActive()` covers both `Active` AND `GraceTimer`
- Immunity NPC: immediate snap (no grace period) from `UpdateStealRadius`

---

## Key Methods

### `IPR_PlayerAbility` Interface (all 7)

| Method | Behavior |
|---|---|
| `CanActivate` | Validates pending target + state + cooldown |
| `Activate_Implementation` | Swaps mesh+tags, calls `createRopeEffect()`, then `BP_OnFaceStealActivated` |
| `Deactivate_Implementation` | Restores mesh+tags, calls `retractRopeEffect()`, then `BP_OnFaceStealDeactivated` |
| `StartTargeting` | Begins trace loop |
| `CancelTargeting` | Aborts trace, fires `BP_OnTargetLost` |
| `IsActive` | Returns true in Active or GraceTimer |
| `GetAbilityState` | Returns `CurrentState` |

### Niagara Rope

| Method | Trigger | Action |
|---|---|---|
| `createRopeEffect()` | `Activate_Implementation` | SpawnSystemAttached, set RopeStart/RopeEnd |
| `retractRopeEffect()` | `Deactivate_Implementation` | DeactivateImmediate, null RopeEffect |
| `snapRopeEffect()` | `OnGraceTimerExpired` | DeactivateImmediate, null RopeEffect |

**TickComponent:** updates `User.RopeStart` + `User.RopeEnd` while State is Active or GraceTimer.

**Distance check is 2D by design:** `FVector::DistSquaredXY(...)` in `UpdateStealRadius` and `UpdateRopeWarning` — height difference does not trigger snap.

---

## Blueprint Events

| Event | Fired when |
|---|---|
| `BP_OnTargetFound` | NPC enters targeting reticle |
| `BP_OnTargetLost` | NPC leaves targeting reticle |
| `BP_OnFaceStealActivated` | After mesh/tag swap + rope created |
| `BP_OnFaceStealDeactivated` | After mesh/tag restore + rope removed |
| `BP_OnRopeSnapped` | Grace timer expired (rope snapped) |
| `BP_OnRopeWarning` | Player approaching radius boundary |
| `BP_OnEnterRadius` | Player returned inside radius |

---

## Jira

| Ticket | Topic | Status |
|---|---|---|
| PR-95 | Core implementation | Done |
| PR-96 | Wire with AI system | In Progress |
| PR-97 | HUD / UI | In Progress |
| PR-98 | VFX | In Progress |
| PR-99 | Sounds | To Do |

---

## Related

- [[game-design/mechanics/FaceSteal]] — mechanic spec
- [[game-design/technical-architecture/FaceSteal_VFX]] — VFX + UI architecture
- [[sessions/2026-06-15_FaceSteal_VFX_UI_Niagara]] — VFX/UI session
- [[sessions/2026-05-16_FaceStealComponent_CPP]] — C++ implementation session
- [[entities/APR_BasePlayer]] — player pawn that hosts this component

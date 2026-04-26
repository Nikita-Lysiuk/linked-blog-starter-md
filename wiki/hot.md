---
type: hot-cache
title: Current Sprint Context
updated: '2026-04-27'
tags:
  - hot
  - sprint
---

# Hot Cache — Ability Sprint

## Current Sprint: Ability Sprint (Sprint 4)

**Goal:** Connect P1 and P2 abilities to the AI system built in AI Sprint. Make the stealth loop actually playable.

## Do Next

1. **Implement `UPR_FaceStealComponent.cpp`** — method bodies per session design
2. **Add tag check to `BTService_PR_UpdateAlertFromSight.cpp`** — `static const FGameplayTag FaceStealActiveTag` at file scope, `HasGameplayTag` check in `TickNode` before alert delta
3. **Remove stale future-method comments** from `PR_AIAbilityTarget.h` (`StealAppearance`, `PasteConsciousness`)

## What Was Just Finished

- `UPR_FaceStealComponent` **header complete** — all 4 fixes applied:
  - Renamed `UFaceStealComponent` → `UPR_FaceStealComponent` / `PR_FaceStealComponent.h`
  - `enum class EFaceStealState : uint8` (was missing `: uint8`)
  - `UENUM(BlueprintType)` (removed invalid `Category` specifier)
  - `ClassGroup=(Nocturne)` (was `Custom`)
- `DefaultGameplayTags.ini` cleaned — 5 tags: `AI.Type.*` (4) + `Ability.FaceSteal.Active`
- `IPR_PlayerAbility` interface finalised — `StartTargeting`, `StopTargeting`, `IsTargeting` (bool), `CanActivate` (no param), `Activate` (no param), `Deactivate`, `IsActive`
- `RuntimeTags` on `APR_BaseCharacter` — confirmed done, `DOREPLIFETIME` in place
- Wiki lint run + all stale pages updated

## Key Design Decisions Locked

- **`UActorComponent`** — no transform needed
- **`EFaceStealState`**: `Inactive`, `Targeting`, `Active`, `GraceTimer`
- **`StealRadius` on component** (not EnemyConfig) — predictability for player
- **No `StealAppearance()` on interface** — component orchestrates via `GetMemoryComponent()` + casts
- **No delegates** — `BlueprintImplementableEvent` hooks are sufficient
- **Montages** for ability reactions, not anim state machine
- **`static const FGameplayTag` at file scope** in `.cpp` — not BT node memory
- **PR-92 cancelled** — `AI.State.*` tags removed, BB handles state
- Two-phase activation: `StartTargeting()` (E) → `Activate()` (LMB)
- `PendingTarget`: `TWeakObjectPtr<AActor>` — cached during targeting tick

## `UPR_FaceStealComponent` Implementation Reference

**Properties (EditDefaultsOnly):** `TraceDistance` (2000cm), `StealRadius` (1500cm), `GraceTimerDuration` (3s), `CooldownDuration` (15s)

**Private state:** `PendingTarget` (TWeakObjectPtr), `StealTarget` (TWeakObjectPtr), `OriginalMesh` (TObjectPtr<USkeletalMesh>), `CurrentState` (EFaceStealState), `GraceTimerHandle`, `CooldownTimerHandle`, `bOnCooldown`

**`Activate_Implementation()` flow:**
1. Validate `PendingTarget.IsValid()`
2. Cast to `IPR_AIAbilityTarget` → `CanBeTargetedByAbility()` → false? immunity snap
3. `GetMemoryComponent()` → `SetConsciousnessState(Frozen)`
4. Cast to `APR_BaseCharacter` → copy `RuntimeTags` to P1
5. Cast to `ACharacter` → store `GetMesh()->GetSkeletalMeshAsset()` as `OriginalMesh`, apply NPC mesh to P1
6. Add `Ability.FaceSteal.Active` tag to P1
7. `StealTarget = PendingTarget`, `CurrentState = Active`
8. Fire `BP_OnFaceStealActivated(Target)`

**`Deactivate_Implementation()` flow:**
1. Restore `OriginalMesh` on P1's mesh component
2. Remove stolen `RuntimeTags` from P1
3. Remove `Ability.FaceSteal.Active` from P1
4. Cast `StealTarget` to `IPR_AIAbilityTarget` → `BeginDisorientation()`
5. Clear `StealTarget`, `CurrentState = Inactive`
6. Start `CooldownTimerHandle` → `OnCooldownExpired`
7. Fire `BP_OnFaceStealDeactivated()`

## Jira Tickets

| Key | Summary | Status |
|-----|---------|--------|
| PR-87 | UPR_FaceStealComponent (P1) | **In Progress — header done, .cpp next** |
| PR-88 | Wire IPR_AIAbilityTarget on APR_BaseAI | To Do |
| PR-89 | P2 Telepathy — Copy | To Do |
| PR-90 | P2 Telepathy — Paste A | To Do |
| PR-91 | P2 Telepathy — Paste B | To Do |
| PR-92 | Apply AI.State.* tags | ❌ Cancelled |
| PR-93 | AI perception tuning | To Do |
| PR-94 | Place LightZone volumes | To Do |

## Full Design Docs
- [[2026-04-27_FaceStealComponent_Design]] — full design session, all requirements
- [[2026-04-26_FaceSteal_Sprint]] — previous session decisions
- [[AbilitySprint_Implementation]] — implementation plan
- [[FaceSteal]] — mechanic spec (updated)

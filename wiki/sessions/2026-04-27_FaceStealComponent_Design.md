---
type: session
title: Session — UFaceStealComponent Full Design
date: '2026-04-27'
updated: '2026-04-27'
tags:
  - session
  - facesteal
  - sprint
  - abilities
  - design
status: complete
---

# Session — UPR_FaceStealComponent Full Design (2026-04-27)

> Resume point. Read this + [[hot]] before starting next session.

---

## What Was Done This Session

| Item | Status |
|------|--------|
| RuntimeTags refactor to `APR_BaseCharacter` | ✅ Confirmed done |
| Tag system redesign | ✅ 5 tags total |
| `BTService_PR_UpdateAlertFromSight` tag caching pattern | ✅ Decided |
| `IPR_PlayerAbility` interface redesign | ✅ Written + reviewed |
| `IPR_AIAbilityTarget` SOLID audit | ✅ Stays as query/access only |
| `UPR_FaceStealComponent` full design | ✅ Locked |
| `UPR_FaceStealComponent` header written | ✅ All 4 fixes applied |
| Wiki lint + stale page updates | ✅ Done |

---

## Header Fixes Applied

| Fix | Before | After |
|-----|--------|-------|
| Class naming | `UFaceStealComponent` | `UPR_FaceStealComponent` |
| Enum declaration | `enum EFaceStealState` | `enum class EFaceStealState : uint8` |
| UENUM specifier | `UENUM(BlueprintType, Category="Face Steal")` | `UENUM(BlueprintType)` |
| ClassGroup | `ClassGroup=(Custom)` | `ClassGroup=(Nocturne)` |

**File:** `Public/Mechanics/Abilities/PR_FaceStealComponent.h`

---

## Key Decisions Made This Session

### 1. Tag system stripped to 2 categories
- `Ability.FaceSteal.Active` — on P1 while steal is active. BT service reads this to suppress AlertLevel delta.
- `AI.Type.Guard/Officer/Civilian/Mech` — NPC identity. Future: Officer secondary validation.
- Everything else deleted: `AI.Faction.*`, `AI.State.*`, `Player.*`, extra ability tags.

### 2. `BTService_PR_UpdateAlertFromSight` — tag caching
`FAlertFromSightMemory` struct removed. Correct pattern — in `.cpp` at file scope:
```cpp
static const FGameplayTag FaceStealActiveTag =
    FGameplayTag::RequestGameplayTag(FName("Ability.FaceSteal.Active"));
```
Check in `TickNode`: if perceived actor has this tag → skip AlertLevel delta.

### 3. `IPR_AIAbilityTarget` — query/access interface only
No `StealAppearance()`. Adding ability-specific methods violates OCP. Component orchestrates directly:
1. `CanBeTargetedByAbility()` → gate
2. `GetMemoryComponent()` → `SetConsciousnessState(Frozen)` directly
3. Cast to `APR_BaseCharacter` → `GetRuntimeTags()` → copy to P1
4. Cast to `ACharacter` → `GetMesh()` → store + swap

### 4. `IPR_PlayerAbility` — final interface
| Method | Trigger |
|--------|---------|
| `StartTargeting()` | E pressed |
| `StopTargeting()` | E pressed again |
| `CanActivate() const` | Cooldown + state check (no param) |
| `Activate()` | LMB — uses cached `PendingTarget` (no param) |
| `Deactivate()` | Grace timer / manual |
| `IsActive() const` | Query |
| `IsTargeting() const` | Query |

### 5. `StealRadius` on component, not EnemyConfig
Predictability: consistent range the player can learn. `UPR_FaceStealComponent::StealRadius = 1500cm`.

### 6. `UPR_FaceStealComponent` full design

**Class:** `UActorComponent`, implements `IPR_PlayerAbility`

**EFaceStealState:** `Inactive`, `Targeting`, `Active`, `GraceTimer`

**Designer properties (EditDefaultsOnly):**
- `TraceDistance` = 2000cm
- `StealRadius` = 1500cm
- `GraceTimerDuration` = 3s
- `CooldownDuration` = 15s

**Private runtime state:**
- `PendingTarget`: `TWeakObjectPtr<AActor>`
- `StealTarget`: `TWeakObjectPtr<AActor>`
- `OriginalMesh`: `TObjectPtr<USkeletalMesh>`
- `CurrentState`: `EFaceStealState`
- `GraceTimerHandle`, `CooldownTimerHandle`: `FTimerHandle`
- `bOnCooldown`: `bool`

**BlueprintImplementableEvents:**
`BP_OnTargetingStarted`, `BP_OnTargetingCancelled`, `BP_OnTargetFound(AActor*)`, `BP_OnTargetLost`, `BP_OnFaceStealActivated(AActor*)`, `BP_OnRopeWarning(float, float)`, `BP_OnEnterRadius`, `BP_OnFaceStealDeactivated`

No Niagara/sound/UMG pointers in C++. No delegates. Montages (not state machine) for animations.

---

## `Activate_Implementation()` Flow
1. `PendingTarget.IsValid()` — bail if invalid
2. Cast to `IPR_AIAbilityTarget` → `CanBeTargetedByAbility()` → false? immunity snap + bail
3. `GetMemoryComponent()` → `SetConsciousnessState(Frozen)`
4. Cast to `APR_BaseCharacter` → copy `RuntimeTags` to P1
5. Cast to `ACharacter` → store mesh asset as `OriginalMesh`, apply NPC mesh to P1
6. `AddGameplayTag(FaceStealActiveTag)` on P1
7. `StealTarget = PendingTarget`, `CurrentState = Active`
8. `BP_OnFaceStealActivated(Target)`

## `Deactivate_Implementation()` Flow
1. Restore `OriginalMesh` on P1's mesh component
2. Remove stolen `RuntimeTags` from P1
3. `RemoveGameplayTag(FaceStealActiveTag)` from P1
4. Cast `StealTarget` to `IPR_AIAbilityTarget` → `BeginDisorientation()`
5. Clear `StealTarget`, `CurrentState = Inactive`
6. Start `CooldownTimerHandle`
7. `BP_OnFaceStealDeactivated()`

---

## Exact Next Steps

1. **Implement `UPR_FaceStealComponent.cpp`**
2. **Add tag check to `BTService_PR_UpdateAlertFromSight.cpp`**
3. **Remove stale comments** from `PR_AIAbilityTarget.h` (`StealAppearance`, `PasteConsciousness`)

---

## Related
- [[AbilitySprint_Implementation]] — implementation plan
- [[FaceSteal]] — mechanic spec (updated this session)
- [[hot]] — current sprint state
- [[meta/lint-report-2026-04-27]] — wiki health check

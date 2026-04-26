---
type: session
title: Session — UFaceStealComponent Full Design
date: '2026-04-27'
tags:
  - session
  - facesteal
  - sprint
  - abilities
  - design
---

# Session — UFaceStealComponent Full Design (2026-04-27)

> Resume point. Read this + [[hot]] before starting next session.

---

## What Was Done This Session

| Item | Status |
|------|--------|
| RuntimeTags refactor to `APR_BaseCharacter` | ✅ Confirmed done (DOREPLIFETIME already in BaseCharacter) |
| Tag system redesign | ✅ Decided — stripped to 2 categories |
| `BTService_PR_UpdateAlertFromSight` tag caching | ✅ Decided — `static const` at file scope, struct removed |
| `IPR_PlayerAbility` interface redesign | ✅ Written by developer, reviewed |
| `IPR_AIAbilityTarget` SOLID audit | ✅ Decided — stays as query/access only, no ability-specific methods |
| `UFaceStealComponent` full design | ✅ Locked |

---

## Key Decisions Made This Session

### 1. Tag system stripped to 2 categories
Old system had ~16 tags. Everything driven by BB or component state doesn't need a tag.

**Keep:**
- `Ability.FaceSteal.Active` — on P1 while steal is active. BT service reads this to suppress AlertLevel delta.
- `AI.Type.Guard`, `AI.Type.Officer`, `AI.Type.Civilian`, `AI.Type.Mech` — NPC identity. Future: Officer secondary validation branch.

**Delete everything else:** `AI.Faction.*`, `AI.State.*` (BB handles state), `Player.*`, `Ability.FaceSteal.Cooldown`, `Ability.Telepathy.*` (add back when building Telepathy).

**Why:** BB handles state, component state handles ability tracking. Tags are only for cross-system identity queries — not internal bookkeeping.

### 2. `BTService_PR_UpdateAlertFromSight` — tag caching pattern
`FAlertFromSightMemory` struct **removed**. Node memory is for per-AI mutable state, not constants.

Correct pattern — in `.cpp` at file scope:
```cpp
static const FGameplayTag FaceStealActiveTag =
    FGameplayTag::RequestGameplayTag(FName("Ability.FaceSteal.Active"));
```
`TickNode` reads it directly. Zero per-AI cost.

Tag check in `TickNode`: if perceived actor has `FaceSteal.Active` tag → skip AlertLevel delta entirely.

### 3. `IPR_AIAbilityTarget` — query/access interface only
**No `StealAppearance()`.** Adding ability-specific methods to this interface violates OCP — every new ability would require modifying it.

Interface responsibility: tell abilities **whether** they can target an AI, and **give access** to AI components. The ability component orchestrates everything itself via `GetMemoryComponent()` + casts.

`FaceStealComponent::Activate()` flow (no interface method needed):
1. Cast `PendingTarget` to `IPR_AIAbilityTarget`
2. `CanBeTargetedByAbility()` → false? immunity snap, done
3. `GetMemoryComponent()` → `SetConsciousnessState(Frozen)` directly
4. Cast to `APR_BaseCharacter` → `GetRuntimeTags()` → copy to P1
5. Cast to `ACharacter` → `GetMesh()` → store + apply NPC mesh to P1

Remove the future method comments (`StealAppearance`, `PasteConsciousness`) from the interface header — they represent the old (wrong) design.

### 4. `IPR_PlayerAbility` interface — final shape
Developer updated the interface. Final method set:

| Method | Trigger | Purpose |
|--------|---------|---------|
| `StartTargeting()` | E pressed | Enter targeting mode, enable trace tick, fire `BP_OnTargetingStarted` |
| `StopTargeting()` | E pressed again | Clear `PendingTarget`, exit targeting, fire `BP_OnTargetingCancelled` |
| `CanActivate() const` | checked before Activate | Cooldown check + not already active. No param — target validity checked inside component during trace tick |
| `Activate()` | LMB pressed | Uses cached `PendingTarget`. No param. |
| `Deactivate()` | Grace timer / manual | Restore state |
| `IsActive() const` | query | |
| `IsTargeting() const` | query | |

**Why no AActor* param on Activate:** `PendingTarget` is cached internally during targeting mode. Passing nullptr was a design smell — the signature was lying.

**Why no param on CanActivate:** checks ability-level conditions (cooldown), not target validity. Target validity is checked inside the component's trace tick via `CanBeTargetedByAbility()`.

### 5. FaceStealRange — single value on component
Range lives on `UFaceStealComponent` (`StealRadius`, `EditDefaultsOnly`), not per-NPC in `EnemyConfig`.

**Why:** Predictability. If each NPC had a different range, players couldn't learn the system — it would feel random and frustrating. Consistent range = learnable.

### 6. `UFaceStealComponent` — full design locked

**Class:** `UActorComponent` (no transform needed — trace from camera, distance check via owner location)

**Internal state enum (UENUM above class):**
```
EFaceStealState: Inactive, Targeting, Active, GraceTimer
```

**Designer properties (EditDefaultsOnly, private + AllowPrivateAccess):**

| Variable | Type | Default |
|----------|------|---------|
| `TraceDistance` | `float` (cm) | `2000.f` |
| `StealRadius` | `float` (cm) | `1500.f` |
| `GraceTimerDuration` | `float` (s) | `3.f` |
| `CooldownDuration` | `float` (s) | `5.f` |

**Private runtime state:**

| Variable | Type |
|----------|------|
| `PendingTarget` | `TWeakObjectPtr<AActor>` |
| `StealTarget` | `TWeakObjectPtr<AActor>` |
| `OriginalMesh` | `TObjectPtr<USkeletalMesh>` |
| `CurrentState` | `EFaceStealState` |
| `GraceTimerHandle` | `FTimerHandle` |
| `CooldownTimerHandle` | `FTimerHandle` |
| `bOnCooldown` | `bool` |

**IPR_PlayerAbility overrides:**

| Method | Body |
|--------|------|
| `CanActivate_Implementation()` | `return !bOnCooldown && CurrentState == Inactive` |
| `StartTargeting_Implementation()` | State → Targeting, fire `BP_OnTargetingStarted` |
| `StopTargeting_Implementation()` | Clear PendingTarget, state → Inactive, fire `BP_OnTargetingCancelled` |
| `Activate_Implementation()` | Validate PendingTarget → freeze NPC → copy tags + mesh → state → Active → fire `BP_OnFaceStealActivated` |
| `Deactivate_Implementation()` | Restore mesh + tags → `BeginDisorientation` on NPC → start cooldown → state → Inactive → fire `BP_OnFaceStealDeactivated` |
| `IsActive_Implementation()` | `return CurrentState == Active \|\| CurrentState == GraceTimer` |
| `IsTargeting_Implementation()` | `return CurrentState == Targeting` |

**Private helpers:**

| Method | Body |
|--------|------|
| `TickComponent` | Targeting: line trace, update PendingTarget, fire BP_OnTargetFound/Lost. Active/GraceTimer: check distance vs StealRadius, manage grace timer |
| `OnGraceTimerExpired()` | `Deactivate_Implementation()` |
| `OnCooldownExpired()` | `bOnCooldown = false` |

Timer callbacks need `UFUNCTION()` (empty specifier) so UE's timer system can find them via reflection.

**BlueprintImplementableEvents (no C++ body, BP wires all assets):**

| Event | When | BP responsibility |
|-------|------|-------------------|
| `BP_OnTargetingStarted()` | StartTargeting | Dark smoke VFX on hands, UMG vignette overlay, sound |
| `BP_OnTargetingCancelled()` | StopTargeting | Remove VFX, hide UI |
| `BP_OnTargetFound(AActor*)` | Trace hits valid NPC | White outline on NPC |
| `BP_OnTargetLost()` | Trace loses NPC | Remove outline |
| `BP_OnFaceStealActivated(AActor*)` | Activate succeeds | Soul-smoke beam NPC→P1, sound, anim montage |
| `BP_OnRopeWarning(float TimeRemaining, float MaxTime)` | Each tick outside radius | Drive rope colour/pulse |
| `BP_OnEnterRadius()` | P1 re-enters radius | Cancel pulse, rope normal |
| `BP_OnFaceStealDeactivated()` | Deactivate | Snap rope VFX, restore UI, break sound |

**No delegates.** BP events are sufficient. No external C++ system subscribes to FaceSteal state yet.

**Animations:** Montages triggered from BP event handlers. Not anim state machine — ability reactions are one-shot events, not continuous locomotion states.

---

## Bugs Found — Fix Before Writing Component

| File | Issue |
|------|-------|
| `PR_PlayerAbility.h` line 31 | `IsTargeting()` returns `void` — must return `bool` |
| `PR_AIAbilityTarget.h` line 62 | Typo `BLueprintNativeEvent` (capital L) — fix to `BlueprintNativeEvent` |

---

## Exact Next Steps (do in this order)

1. **Fix `IsTargeting()` return type** in `PR_PlayerAbility.h` (void → bool)
2. **Fix `BLueprintNativeEvent` typo** in `PR_AIAbilityTarget.h` line 62
3. **Remove future method comments** (`StealAppearance`, `PasteConsciousness`) from `PR_AIAbilityTarget.h`
4. **Clean up `DefaultGameplayTags.ini`** — delete all except `Ability.FaceSteal.Active` + `AI.Type.*` block
5. **Write `UFaceStealComponent` header** — `EFaceStealState` enum + class declaration per requirements above
6. **Implement `UFaceStealComponent.cpp`** — method bodies per table above

---

## What's Still Open

- `BTService_PR_UpdateAlertFromSight` .cpp needs the `static const` tag + `HasGameplayTag` check added — not yet implemented
- `IPR_AIAbilityTarget` future method comments still need removal from header
- PR-92 ("Apply AI.State.* tags") is effectively cancelled — `AI.State.*` tags deleted, BB handles state. Can be closed.

---

## Related
- [[AbilitySprint_Implementation]] — implementation plan (needs PR-92 cancellation noted)
- [[FaceSteal]] — mechanic spec
- [[hot]] — current sprint state

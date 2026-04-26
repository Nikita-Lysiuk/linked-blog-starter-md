---
type: session
title: "Session — FaceSteal Sprint Kickoff"
date: '2026-04-26'
tags:
  - session
  - facesteal
  - sprint
  - abilities
---

# Session — FaceSteal Sprint Kickoff (2026-04-26)

> Resume point. Read this + [[hot]] before starting next session.

---

## What Was Built This Session

| Item | Status |
|------|--------|
| `BTTask_PR_Disorient` | ✅ Written, reviewed, bugs fixed |
| `IPR_PlayerAbility` interface | ✅ Written (4 methods) |
| `AbilityComponent` on `APR_BasePlayer` | ✅ Added (`TScriptInterface<IPR_PlayerAbility>`, `EditDefaultsOnly`, private + AllowPrivateAccess) |
| `BeginDisorientation()` on `IPR_AIAbilityTarget` | ✅ Designed — `BlueprintNativeEvent`, empty C++ body, BP plays animation |
| `TriggerDisorientation()` on `IPR_AIAbilityTarget` | ✅ Designed — sets `bIsDisoriented = true` on own Blackboard |
| `bIsDisoriented` BB key | ✅ Added to Blackboard asset |

---

## Key Decisions Made This Session

### 1. Overwritten state removed
`EAIConsciousnessState::Overwritten` is deleted. Replaced by:
- `bIsDisoriented` Blackboard bool key
- `BTTask_PR_Disorient` (written this session) handles the 2-second wake-up window
- `BTTask_PR_HandleOverwritten` is deleted (was doing the same job)
- `OverwrittenEndTime` BB key is deleted

**Enum is now:** `Normal`, `Frozen`, `Replaced` only.

### 2. Team ID NEVER changes for disguise
Previous design had P1's team ID switching 0→1 on FaceSteal. **Dropped entirely.**
- `APR_PlayerController::GetGenericTeamId()` stays hardcoded at 0
- AI perception will technically still fire on P1
- Disguise works through **GameplayTag check only** — if P1 has matching NPC tags, a BT service suppresses AlertLevel increase
- More flexible for Act 2 paranoid officers (tag-based, not binary team check)

### 3. ApplyPasteA / ApplyPasteB removed from APR_AIController
These are ability entry points — they don't belong on the controller. Moving to ability components via the interface. What stays on the controller:
- `RestoreFromSnapshot()` — BT task callback, touches BB
- `RestoreFromOwnStateSnapshot()` — BT task callback, touches BB
- `ApplySnapshotToBB()` — private BB write helper

### 4. RuntimeTags moving to APR_BaseCharacter
Currently only on `APR_BaseAI`. Both player and AI need readable tags for the disguise tag-check system.
**Refactor:** Move `RuntimeTags`, `GetRuntimeTags()`, `HasGameplayTag()`, `AddGameplayTag()`, `RemoveGameplayTag()` from `APR_BaseAI` → `APR_BaseCharacter`. Keep `InitializeRuntimeTags()` in `APR_BaseAI` (reads from EnemyConfig).

### 5. FaceSteal is first-person
Only third-person body mesh swaps (visible to P2 and NPCs). First-person hands never change. P1 doesn't see their own disguise.

### 6. FaceSteal breach timer is resettable
Exit radius → timer starts. Re-enter radius → timer resets. Player can go in and out, but has a limited window to return each time. Not a one-way countdown.

### 7. NPC death = bug, not a feature
NPCs cannot die in this game. NPC becoming invalid mid-steal is a bug to be avoided in level design, not an edge case to handle in code. Component should still null-check the reference (assert/log), but no graceful recovery needed.

### 8. IPR_PlayerAbility interface design
```cpp
bool CanActivate(AActor* Target) const;
void Activate(AActor* Target);       // AActor* — reticle hands back an Actor
void Deactivate();
bool IsActive() const;
```
`APR_BasePlayer::AbilityComponent` is `TScriptInterface<IPR_PlayerAbility>`. BP_P1 assigns `UFaceStealComponent`, BP_P2 assigns `UTelepathyComponent`.

---

## FaceSteal Full State Machine (locked)

| State | Trigger | What happens |
|---|---|---|
| `Inactive` | — | Nothing |
| `Targeting` | Button press | Screen darkens + vignette, center glow VFX, line trace begins |
| `Active` | LMB confirms NPC | NPC → Frozen, mesh swapped, tags mirrored, rope appears (green) |
| `BreachWarning` | P1 exits radius | Rope flashes red, grace timer starts (resettable) |
| `Deactivating` | Grace timer expires or manual cancel | Rope snaps, mesh restored, tags cleared, NPC → TriggerDisorientation |

### Targeting Mode VFX (Dishonored-style)
- Dark smoke on screen edges
- Screen darkens except centre
- Glowing circle/reticle in centre
- Line trace from centre of screen begins
- Trace length = configurable on `UFaceStealComponent`
- Player looks at NPC + LMB → activates

### Rope visual
- Green → yellow → red as player approaches radius edge
- Flashes when outside radius (Fresnel/pulse effect)
- Snaps after grace timer

---

## Exact Next Steps (do in this order)

1. **Delete `Overwritten` from `EAIConsciousnessState`** (PR_AITypes.h)
2. **Delete `BTTask_PR_HandleOverwritten`** (header + cpp)
3. **Delete `OverwrittenEndTime` BB key** from PR_AIBlackboardKeys.h
4. **Delete `ApplyPasteA` / `ApplyPasteB`** from `APR_AIController` (header + cpp)
5. **Refactor RuntimeTags → `APR_BaseCharacter`** (move from APR_BaseAI)
6. **Add `BeginDisorientation()` + `TriggerDisorientation()` to `IPR_AIAbilityTarget`** (header only — implementations go on APR_BaseAI)
7. **Wire `BTTask_PR_Disorient` + `bIsDisoriented` decorator into BT asset**
8. **Write `UFaceStealComponent`** — implements `IPR_PlayerAbility`
9. **Write `StealAppearance()` on `IPR_AIAbilityTarget`** — NPC side (sets Frozen state)

---

## What's Still Open

- `BTService_PR_UpdateAlertFromSight` needs a tag check: if perceived actor has matching faction tags, suppress AlertLevel delta. This is the mechanism that makes disguise actually work. Not yet implemented.
- Target selection in `UFaceStealComponent`: line trace from camera centre, trace length = component parameter
- Rope: spline mesh or Niagara beam between P1 and NPC
- Sound: 3 trigger points (activation, loop, snap)

---

## Related
- [[FaceSteal]] — mechanic spec (updated this session)
- [[AbilitySprint_Implementation]] — implementation plan (updated this session)
- [[hot]] — current sprint state

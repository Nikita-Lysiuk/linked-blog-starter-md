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

1. **Fix 4 issues in `PR_FaceStealComponent.h`** (rename from FaceStealComponent.h):
   - Rename class/file: `UFaceStealComponent` → `UPR_FaceStealComponent`
   - `enum EFaceStealState` → `enum class EFaceStealState : uint8` (UHT requirement)
   - `UENUM(BlueprintType, Category="Face Steal")` → `UENUM(BlueprintType)` (Category invalid here)
   - `ClassGroup=(Custom)` → `ClassGroup=(Nocturne)`
2. **Add tag check to `BTService_PR_UpdateAlertFromSight.cpp`** — `static const FGameplayTag` at file scope, `HasGameplayTag` check in `TickNode` before alert delta
3. **Remove stale comments** from `PR_AIAbilityTarget.h` — delete `StealAppearance` / `PasteConsciousness` future-method block
4. **Implement `UPR_FaceStealComponent.cpp`**

## What Was Just Finished

- `DefaultGameplayTags.ini` cleaned — 5 tags total: `AI.Type.*` (4) + `Ability.FaceSteal.Active`
- `IPR_PlayerAbility` updated — `StartTargeting`, `StopTargeting`, `IsTargeting` (bool), `CanActivate` (no param), `Activate` (no param), `Deactivate`, `IsActive`
- `IsTargeting()` void→bool bug fixed
- `UFaceStealComponent` header written (`Public/Mechanics/Abilities/FaceStealComponent.h`)
- RuntimeTags on `APR_BaseCharacter` — confirmed done, `DOREPLIFETIME` in place

## Key Design Decisions Locked

- **`UActorComponent`** — no transform needed
- **`EFaceStealState`**: Inactive, Targeting, Active, GraceTimer
- **`StealRadius` on component** (not EnemyConfig) — predictability for player
- **No `StealAppearance()` on interface** — FaceStealComponent orchestrates via `GetMemoryComponent()` + casts
- **No delegates** — `BlueprintImplementableEvent` hooks sufficient
- **Montages** for ability animations, not state machine
- **`static const FGameplayTag` at file scope** in `.cpp` — not node memory
- **PR-92 cancelled** — `AI.State.*` tags gone, BB handles state

## Jira Tickets

| Key | Summary | Status |
|-----|---------|--------|
| PR-87 | UPR_FaceStealComponent (P1) | **In Progress — header done** |
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
- [[FaceSteal]] — mechanic spec

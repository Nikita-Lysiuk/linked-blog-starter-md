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

## Do Next (strict order)

1. **Fix `IsTargeting()` return type** — `PR_PlayerAbility.h` line 31: `void` → `bool`
2. **Fix `BLueprintNativeEvent` typo** — `PR_AIAbilityTarget.h` line 62: capital L
3. **Remove stale comments** — delete `StealAppearance` / `PasteConsciousness` future-method block from `PR_AIAbilityTarget.h`
4. **Clean `DefaultGameplayTags.ini`** — keep only `Ability.FaceSteal.Active` + `AI.Type.*` block, delete everything else
5. **Write `UFaceStealComponent` header** — `EFaceStealState` enum + class per [[2026-04-27_FaceSteal_Sprint|session note]]
6. **Implement `UFaceStealComponent.cpp`**
7. **Add tag check to `BTService_PR_UpdateAlertFromSight.cpp`** — `static const FGameplayTag` at file scope, `HasGameplayTag` check in `TickNode` before alert delta

## What Was Just Finished (this session)

- RuntimeTags refactor: `APR_BaseCharacter` now owns container + CRUD. `APR_BaseAI` keeps `InitializeRuntimeTags()` only.
- `IPR_PlayerAbility` updated: `StartTargeting`, `StopTargeting`, `IsTargeting`, `CanActivate` (no param), `Activate` (no param), `Deactivate`, `IsActive`
- `UFaceStealComponent` fully designed — all properties, methods, BP events locked
- Tag system stripped: ~16 tags → 2 categories (`Ability.FaceSteal.Active` + `AI.Type.*`)
- SOLID audit of `IPR_AIAbilityTarget`: no ability-specific methods — FaceStealComponent orchestrates via `GetMemoryComponent()` + casts

## Key Design Decisions Locked

- **FaceStealRange on component, not EnemyConfig** — predictability for player. Single `StealRadius` float on `UFaceStealComponent`.
- **No `StealAppearance()` on interface** — OCP violation. Component calls `GetMemoryComponent()->SetConsciousnessState(Frozen)` directly.
- **No delegates** — `BlueprintImplementableEvent` hooks are sufficient. No external C++ subscriber yet.
- **Montages, not state machine** — ability reactions are one-shot events, not locomotion blends.
- **`static const FGameplayTag` at file scope** — not in BT node memory. Node memory is for per-AI mutable state, not constants.
- **PR-92 cancelled** — `AI.State.*` tags removed. BB handles state. Ticket can be closed.
- Rope break → AlertLevel stays 0, 1–2s disorientation (Act 1)
- FaceSteal re-use on same NPC: allowed
- Replaced state runs normal BT with donor BB data

## Jira Tickets

| Key | Summary | Status |
|-----|---------|--------|
| PR-87 | UFaceStealComponent (P1) | **In Progress** |
| PR-88 | Wire IPR_AIAbilityTarget on APR_BaseAI | To Do |
| PR-89 | P2 Telepathy — Copy | To Do |
| PR-90 | P2 Telepathy — Paste A | To Do |
| PR-91 | P2 Telepathy — Paste B | To Do |
| PR-92 | Apply AI.State.* tags | ❌ Cancelled — AI.State.* deleted |
| PR-93 | AI perception tuning | To Do |
| PR-94 | Place LightZone volumes | To Do |

## Full Design Docs
- [[2026-04-27_FaceSteal_Sprint]] — today's session, full UFaceStealComponent requirements
- [[AbilitySprint_Implementation]] — implementation plan
- [[FaceSteal]] — mechanic spec
- [[GDD_Nocturne]] — full game design

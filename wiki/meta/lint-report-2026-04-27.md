---
type: meta
title: Lint Report 2026-04-27
created: '2026-04-27'
updated: '2026-04-27'
tags:
  - meta
  - lint
status: active
---

# Lint Report: 2026-04-27

## Summary
- Pages scanned: 22
- Issues found: 18
- Auto-fixed: 0
- Needs review: 18

---

## 🔴 Critical — Duplicate Pages

### 1. `Disguise Steal.md` vs `FaceSteal.md`
Both pages cover the P1 FaceSteal mechanic. They were written at different times and have diverged significantly. `FaceSteal.md` is newer and more detailed. `Disguise Steal.md` is staler and contradicts current design.

**Suggest:** Merge into `FaceSteal.md` (keep the newer), delete `Disguise Steal.md`, update all inbound links.

### 2. `Ability System Interface.md` vs `Universal Ability System Interface.md`
Both files have **identical content** — the same ADR body about BT-driven state vs custom ability manager. One was likely renamed and the original not deleted.

**Suggest:** Delete `Universal Ability System Interface.md`, keep `Ability System Interface.md`. Update any links pointing to the deleted file.

---

## 🔴 Stale Claims — Consciousness State Machine

**File:** `wiki/game-design/technical-architecture/Consciousness State Machine.md`

The `Overwritten` state was **deleted from the codebase** in the 2026-04-26 session. This page still documents it as live.

| Claim | Reality |
|-------|---------|
| `EAIConsciousnessState` has `Overwritten` | Deleted — enum is now `Normal`, `Frozen`, `Replaced` only |
| `BTTask_PR_HandleOverwritten` runs disorientation | Deleted — replaced by `BTTask_PR_Disorient` + `bIsDisoriented` BB key |
| `ApplyPasteA()` / `ApplyPasteB()` on `APR_AIController` | Deleted — moved to ability components |
| `OverwrittenEndTime` BB key referenced | Deleted |
| State transition diagram shows `Overwritten` branch | Outdated — remove the Overwritten branch |

**Suggest:** Rewrite the States section and diagram. Replace `Overwritten` with `BTTask_PR_Disorient` path. Update Paste A/B entry point to ability components.

---

## 🔴 Stale Claims — Disguise Steal / FaceSteal

**File:** `wiki/game-design/mechanics/Disguise Steal.md`

| Claim | Reality |
|-------|---------|
| "No mesh swap. No `UDisguiseComponent`." | **Wrong** — third-person mesh DOES swap (decided 2026-04-26) |
| `FaceStealRange` in `UPR_EnemyConfig` | **Wrong** — range moved to `UPR_FaceStealComponent` as `StealRadius` |
| "Exiting range: tags removed, NPC stays Frozen. Re-entering: tags restored." | **Wrong** — grace timer starts on exit, ability deactivates on expiry. No mid-range tag toggle. |
| `APR_BaseAI::RuntimeTags` at `PR_BaseAI.h:196` | **Wrong** — now on `APR_BaseCharacter` |
| Tag methods at `PR_BaseAI.h:122–133` | **Wrong** — now on `APR_BaseCharacter` |
| `AI.Faction.Mansion` tag referenced | Never existed — faction tags deleted entirely |
| Open Question: "activation component not yet implemented" | Being implemented now (PR-87) |

**File:** `wiki/game-design/mechanics/FaceSteal.md`

| Claim | Reality |
|-------|---------|
| `Player.P1.Disguised` tag applied on activation | Deleted — tag system stripped |
| `AI.State.Frozen` tag applied to NPC | Deleted — state tracked via BB only |
| `FaceStealRange` in `UPR_EnemyConfig` balance table | Moved to component as `StealRadius` |
| Immunity triggers `Overwritten` + `BTTask_PR_HandleOverwritten` | Deleted — disorientation via `BTTask_PR_Disorient` |
| `Player.P1.Disguised` removed on deactivation | Tag doesn't exist anymore |
| Tick interval "0.5s" hardcoded in description | Implementation detail — don't document tick rate in design spec |

---

## 🔴 Stale Claims — APR_BaseAI Entity Page

**File:** `wiki/entities/APR_BaseAI.md`

| Claim | Reality |
|-------|---------|
| `RuntimeTags` property at `PR_BaseAI.h:196` | Moved to `APR_BaseCharacter` |
| `GetRuntimeTags()`, `HasGameplayTag()`, `AddGameplayTag()`, `RemoveGameplayTag()` listed as APR_BaseAI methods | All moved to `APR_BaseCharacter` |

**Suggest:** Move the Identity/Tag Methods section to an `APR_BaseCharacter` entity page. Leave a "inherited from `APR_BaseCharacter`" note on this page.

---

## 🟡 Stale Claims — Ability Interface Pages

**File:** `wiki/game-design/technical-architecture/Ability System Interface.md`

| Claim | Reality |
|-------|---------|
| `ApplyPasteA()` / `ApplyPasteB()` on `APR_AIController` in architecture diagram | Deleted |
| `BTTask_PR_HandleOverwritten` in BT subtree | Deleted |
| `> [!gap] StealAppearance() planned` | Explicitly decided NOT to add — SOLID violation |

**File:** `wiki/entities/IPR_AIAbilityTarget.md`

| Claim | Reality |
|-------|---------|
| "Planned: `StealAppearance()` method — noted in `PR_AIAbilityTarget.h:25–27`" | Decided against — interface stays as query/access only |

---

## 🟡 Stale Claims — UPR_EnemyConfig

**File:** `wiki/entities/UPR_EnemyConfig.md`

| Claim | Reality |
|-------|---------|
| `FaceStealRange` at lines 193–196: "Max distance P1 can maintain stolen appearance" | Range concept moved to `UPR_FaceStealComponent::StealRadius`. `FaceStealRange` in EnemyConfig may still exist but is no longer the authoritative range for the ability. |
| Usage example: `EnemyConfig->FaceStealRange // → P1 range check` | No longer correct — component uses its own `StealRadius` |

---

## 🟡 Stale Claims — APR_BasePlayer Entity Page

**File:** `wiki/entities/APR_BasePlayer.md`

| Claim | Reality |
|-------|---------|
| "Concrete player abilities live in Blueprint subclasses" | Abilities now live in C++ components (`UPR_FaceStealComponent`) assigned in BP |
| `AbilityComponent` not listed in Key Properties | Should be listed: `TScriptInterface<IPR_PlayerAbility>`, `EditDefaultsOnly`, line ~100 |

---

## 🟡 Missing Pages

| Missing Page | Referenced In | Suggest |
|---|---|---|
| `APR_BaseCharacter` | `APR_BaseAI.md`, `APR_BasePlayer.md` — both inherit from it; `RuntimeTags` now lives here | Create entity page |
| `UPR_FaceStealComponent` | `FaceSteal.md`, `hot.md`, session notes | Create entity page after PR-87 header is finalised |
| `IPR_PlayerAbility` | `APR_BasePlayer.md` (implied), session notes | Create interface entity page |

---

## 🟡 Dead Links (Unverified — Require Directory Check)

These links appear in `wiki/index.md` but their target directories were not fully scanned:

| Link | Status |
|------|--------|
| `[[sources/Reference Library]]` | Unverified — `sources/` directory not scanned |
| `[[sources/_index]]` | Unverified |
| `[[patterns/_index]]` | Unverified |
| `[[ecs/_index]]` | Unverified |
| `[[ecs/entities/_index]]` | Unverified |
| `[[concepts/_index]]` | Unverified |
| `[[concepts/Collisions]]` | Unverified |

---

## 🟢 Frontmatter Gaps

| Page | Missing Fields |
|------|----------------|
| `wiki/entities/APR_BaseAI.md` | `status` |
| `wiki/entities/APR_BasePlayer.md` | `status` |
| `wiki/entities/IPR_AIAbilityTarget.md` | `status` |
| `wiki/entities/UPR_EnemyConfig.md` | `status` |
| `wiki/sessions/2026-04-27_FaceStealComponent_Design.md` | `status` |

---

## 🟢 Index Count Mismatch

`wiki/index.md` Catalog table shows:
- "Decisions (ADRs): 0" — but 2 ADR-type pages exist in `game-design/technical-architecture/` (`Ability System Interface.md`, `Universal Ability System Interface.md`). These should either move to `wiki/decisions/` or the count should reflect them.
- "Entities: 6" — correct (`_index.md` excluded)
- "Game Design: 7" — verify after deleting the `Universal Ability System Interface.md` duplicate

---

## Recommended Fix Order

1. **Delete** `Universal Ability System Interface.md` — pure duplicate
2. **Update** `Consciousness State Machine.md` — Overwritten removal is the most critical stale page
3. **Update** `Disguise Steal.md` or **delete and redirect** to `FaceSteal.md`
4. **Update** `FaceSteal.md` — remove deleted tags, fix range source, fix immunity path
5. **Update** `APR_BaseAI.md` — RuntimeTags section moves to BaseCharacter page
6. **Create** `APR_BaseCharacter.md` entity page
7. **Update** `Ability System Interface.md` — remove Overwritten from diagram, remove StealAppearance gap note
8. **Update** `IPR_AIAbilityTarget.md` — remove StealAppearance planned note
9. **Update** `UPR_EnemyConfig.md` — clarify FaceStealRange vs StealRadius
10. **Update** `APR_BasePlayer.md` — add AbilityComponent, fix ability ownership description
11. **Create** `UPR_FaceStealComponent.md` and `IPR_PlayerAbility.md` after PR-87 is complete

Should I auto-fix any of the safe items (frontmatter gaps, duplicate deletion), or do you want to review each one first?

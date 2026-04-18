---
type: mechanic
title: Disguise Steal (FaceSteal)
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - mechanic
  - sprint
  - persival
  - stealth
  - p1
status: implementing
priority: 1
loop_tier: core
related:
  - '[[IPR_AIAbilityTarget]]'
  - '[[APR_BaseAI]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[UPR_EnemyConfig]]'
  - '[[Consciousness State Machine]]'
  - '[[Mind Copy]]'
  - '[[game-design/mechanics/_index]]'
sources: []
---

# Disguise Steal (FaceSteal)

**Project:** Persival — P1 ability
**Loop Tier:** Core
**Status:** `implementing` — infrastructure complete, player-facing activation component TBD

---

## Design Intent

P1 targets a guard and steals their **identity tags** — not their mesh, their *faction identity*. The AI world runs on `GameplayTags`. P1 injects the target NPC's `RuntimeTags` (e.g. `AI.Type.Guard`) into their own tag container; every NPC's perception query then returns "friendly". The fear is not of being *seen*, but of being *recognised*.

The stolen NPC is **frozen in place** for the duration — a live prop. This creates a tight spatial tension: stay within `FaceStealRange` or lose cover.

---

## Mechanism (code-accurate)

### Activation
1. P1 targets an NPC — must implement `IPR_AIAbilityTarget` with `CanBeTargetedByAbility() == true`.
2. NPC's `ConsciousnessState` → **`Frozen`** via `UPR_AIMemoryComponent::SetConsciousnessState()`.
3. `BTTask_PR_HandleFrozen` runs — holds the NPC indefinitely until state changes externally.
4. NPC's `RuntimeTags` (e.g. `AI.Type.Guard`, faction tags) are **copied onto P1's tag container**.
5. AI perception queries read P1's tags → P1 reads as a friendly guard.

### Range Constraint
- P1 must remain within **`FaceStealRange`** (float, cm — configured per-enemy in `UPR_EnemyConfig`).
- **Exiting range:** tags removed from P1. NPC remains `Frozen`.
- **Re-entering range:** tags restored to P1.

### Deactivation
- Manual cancel — OR — new target is selected (overwrites previous).
- NPC's `ConsciousnessState` → **`Normal`**. `BTTask_PR_HandleFrozen` is aborted externally.
- NPC resumes its Behavior Tree from the root.

### Immunity
- Mechanical enemies set `bCanBeTargetedByAbility = false` in their `UPR_EnemyConfig` DataAsset.
- `IPR_AIAbilityTarget::CanBeTargetedByAbility()` gates access — ability fails silently on immune targets.

---

## Inputs & Outputs

| Input | Output |
|-------|--------|
| P1 activates on valid NPC within FaceStealRange | NPC frozen; P1 gains NPC's RuntimeTags |
| P1 exits FaceStealRange | Tags removed from P1; NPC stays Frozen |
| P1 re-enters FaceStealRange | Tags restored to P1 |
| P1 cancels or selects new target | Tags removed; NPC returns to Normal |
| Target has `bCanBeTargetedByAbility = false` | Ability fails silently |

---

## Why Tags, Not Mesh Swap

AI perception in Nocturne identifies friend/foe via **GameplayTags** (`AI.Type.Guard`, `AI.Faction.Mansion`), not via skeletal mesh or class type. P1 literally *becomes* the tag set the NPC system trusts — no perception override hooks needed, no material swaps, no `UDisguiseComponent`.

Implication: two NPCs sharing the same tags are perceptually indistinguishable. P1 wearing either identity works identically from the AI's perspective. Visual mesh difference is cosmetic only.

> [!note] Tag system source
> `APR_BaseAI::RuntimeTags` — `FGameplayTagContainer`, replicated (`PR_BaseAI.h:196`).
> Initialized from `UPR_EnemyConfig::DefaultTags` at `BeginPlay` (server-only). Clients updated via `OnRep_RuntimeTags`.

---

## Technical Implementation

| Concern | Source |
|---------|--------|
| Interface boundary | [[IPR_AIAbilityTarget]] |
| State lock | [[UPR_AIMemoryComponent]]`::SetConsciousnessState(Frozen)` |
| BT lock task | `BTTask_PR_HandleFrozen` — returns `InProgress` indefinitely |
| Range config | `UPR_EnemyConfig::FaceStealRange` — `PR_EnemyConfig.h:193–196` |
| Tag source | `APR_BaseAI::GetRuntimeTags()` — `PR_BaseAI.h:122` |
| Tag manipulation | `AddGameplayTag` / `RemoveGameplayTag` — `PR_BaseAI.h:128–133` |
| Frozen state definition | `EAIConsciousnessState::Frozen` — `PR_AITypes.h:21` |

**No mesh swap. No `UDisguiseComponent`. No `IAbilitySystem`. No duration timer or cooldown in current code.**

---

## Open Questions / Edge Cases

> [!gap] Player-facing activation component not yet implemented
> The P1-side component that calls `AddGameplayTag` on the player and `SetConsciousnessState(Frozen)` on the NPC has not been written. Infrastructure (interface, frozen task, tag system) is complete.

> [!question] P2 Paste A on a Frozen NPC?
> `SetConsciousnessState(Normal)` from Paste A restore would break P1's disguise mid-use. Interaction between abilities not yet specified — needs a cooperative design pass.

> [!question] Two NPCs with identical RuntimeTags?
> Wearing either identity is functionally equivalent. Needs a level-design guideline for guard roles.

---

## Loop Integration

**Feeds into:** Core stealth loop — social infiltration
**Enables:** Passing guard checkpoints, blending into patrols, P1+P2 cooperative distraction
**Depends on:** [[APR_BaseAI]] tag system, [[Consciousness State Machine]], [[IPR_AIAbilityTarget]]

---

## Acceptance Criteria

- [x] Tag system implemented (`APR_BaseAI::RuntimeTags`, replicated)
- [x] `Frozen` state + `BTTask_PR_HandleFrozen` implemented
- [x] `IPR_AIAbilityTarget` interface implemented on `APR_BaseAI`
- [x] `bCanBeTargetedByAbility` immunity flag on `UPR_EnemyConfig`
- [ ] P1 ability activation component written
- [ ] Tag copy/restore on enter/exit FaceStealRange
- [ ] FaceStealRange debug visualisation (sphere)
- [ ] NPC correctly returns to Normal on deactivation
- [ ] Mechanical enemy immunity tested
- [ ] P1+P2 interaction edge case defined and tested

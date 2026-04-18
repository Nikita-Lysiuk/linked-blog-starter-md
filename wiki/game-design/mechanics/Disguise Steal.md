---
type: mechanic
title: "Disguise Steal"
created: 2026-04-18
updated: 2026-04-18
tags:
  - mechanic
  - sprint
  - persival
  - stealth
status: designed
priority: 1
loop_tier: core
related:
  - "[[Universal Ability System Interface]]"
  - "[[Ability System Interface]]"
  - "[[game-design/mechanics/_index]]"
sources: []
---

# Disguise Steal

**Project:** Persival
**Loop Tier:** Core
**Status:** `designed` — awaiting prototype

---

## Design Intent

The player can target a guard within range and "steal" their appearance — copying their skeletal mesh and material overrides to disguise themselves as that specific guard. While disguised, enemy AI perception treats the player as a neutral or allied guard until the disguise breaks.

The mechanic creates **high-tension identity games**: you are the enemy. The fear is not of being seen, but of being *recognized*.

---

## Inputs & Outputs

| Input | Output | Feedback |
|-------|--------|----------|
| Player activates ability on targeted guard | Player mesh swapped to guard's mesh + materials | Visual shimmer transition effect |
| Guard is incapacitated or dead | Disguise source available | Contextual prompt appears |
| Player attacks while disguised | Disguise instantly broken | Flash + alarm state triggered |
| Cooldown expires | Disguise expires | Shimmer fade-out, player mesh restores |
| Player re-activates or presses cancel | Disguise cancelled early | Same fade-out animation |

---

## Rules

1. Target must be a valid `AGuardCharacter` within `DisguiseRange` (default: 200 units).
2. The guard must be incapacitated, dead, or in a specific social state (e.g., `Idle`, `Patrol`) — not in combat or on alert.
3. Disguise duration is capped by `DisguiseDuration` (default: 60s) — visible timer in HUD.
4. Any hostile action (attack, vault, sprint above threshold) breaks the disguise.
5. Guards with `BDisguiseAware` flag enabled will detect disguise at close range.
6. Cooldown begins on cancellation or expiry, not on activation.

## Edge Cases

- What happens when the source guard is revived while player is disguised? → Disguise should break (two of the same guard = immediate suspicion).
- What if the player dies while disguised? → Mesh restores to default on respawn.
- Multiple guards of same archetype (identical mesh)? → Disguise is still per-instance (tracks the specific guard actor).

> [!gap] Disguise Detection Radius
> No design decision yet on whether `BDisguiseAware` guards detect based on distance only or also player behavior (running, weapon drawn). File ADR when decided.

---

## Loop Integration

**Feeds into:** [[game-design/loop-logic/_index]] — Core stealth loop
**Enables:** Social navigation, guard manipulation, infiltration paths
**Depends on:** AI Perception system (AIPerceptionComponent), Guard state machine

> [!contradiction] Disguise vs. Alert State
> If a guard goes into combat-alert while player is wearing their disguise, does the disguise break? Current design says "alert state breaks disguise" — but this hasn't been reconciled with the guard revive edge case above. Both rules need to be unified in one state machine.

---

## Technical Implementation

→ [[Universal Ability System Interface]]
→ [[Ability System Interface]]

**UE5 Systems involved:**
- `UAbilitySystemComponent` (GAS) or custom `IAbilitySystem` interface
- `USkeletalMeshComponent` — mesh + material swap
- `UAIPerceptionSystem` — faction/perception override while disguised
- `UMaterialInstanceDynamic` — runtime material override per-slot

**ECS / Actor Components required:**
- `UDisguiseComponent` — holds stolen mesh ref, duration timer, break conditions
- `UStealthStatusComponent` — tracks disguise state, AI awareness level

**Patterns used:**
- [[Observer]] — notify AI perception system on disguise state change
- [[Ability System Interface]] — IAbilitySystem::ExecuteAbility, CancelAbility, Cooldown

---

## Acceptance Criteria

- [ ] Design spec complete ✓
- [ ] [[Universal Ability System Interface]] note linked ✓
- [ ] `UDisguiseComponent` implemented in C++
- [ ] Mesh/material swap working in prototype
- [ ] AI perception correctly overridden during disguise
- [ ] Cooldown and duration timers functional
- [ ] All break conditions tested (attack, revival, alert)
- [ ] HUD timer integrated

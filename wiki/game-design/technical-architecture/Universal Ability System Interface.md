---
type: decision
title: "Universal Ability System Interface"
created: 2026-04-18
updated: 2026-04-18
tags:
  - technical-architecture
  - adr
  - persival
  - ability-system
  - sprint
status: proposed
priority: 1
related:
  - "[[Ability System Interface]]"
  - "[[Disguise Steal]]"
  - "[[game-design/technical-architecture/_index]]"
sources: []
---

# Universal Ability System Interface

**Project:** Persival
**Scope:** All hero abilities in the game — Disguise Steal and all future abilities
**Status:** Proposed — pending prototype validation

---

## Problem

Persival will have multiple hero abilities (Disguise Steal, more TBD). Without a unified interface, each ability becomes a bespoke implementation with no shared lifecycle contract. This leads to:
- Duplicate cooldown logic per ability
- No common cancel/interrupt path
- Difficult to add new abilities without touching existing systems
- No ability to stack, queue, or composite abilities at runtime

---

## Solution: IAbilitySystem Contract

Every hero ability in Persival implements `IAbilitySystem` (see [[Ability System Interface]]). The interface enforces a minimal lifecycle contract:

```
Execute → [Active] → Cancel | Expire → Cooldown → Ready
```

The system is **intentionally minimal**. If GAS (Unreal's Gameplay Ability System) is adopted later, this interface becomes an adapter layer — not a replacement.

---

## Architecture Overview

```
APlayerCharacter
└── UAbilityManagerComponent
    ├── TArray<TScriptInterface<IAbilitySystem>> ActiveAbilities
    ├── GrantAbility(TScriptInterface<IAbilitySystem>)
    ├── ActivateAbility(int32 SlotIndex, AActor* Instigator)
    └── CancelAllAbilities()

IAbilitySystem  ← [[Ability System Interface]]
    ├── ExecuteAbility(AActor* Instigator)
    ├── CancelAbility()
    ├── GetCooldownRemaining() const
    └── IsOnCooldown() const

UDisguiseAbility : UObject, IAbilitySystem  ← [[Disguise Steal]]
UFutureAbility   : UObject, IAbilitySystem  ← (future)
```

---

## Design Decisions

### 1. Custom interface vs. Unreal GAS
| Approach | Pros | Cons |
|----------|------|------|
| Custom `IAbilitySystem` | Lightweight, full control, no GAS boilerplate | Manual replication, no prediction |
| Unreal GAS | Industry-standard, replication built-in | High learning curve, large footprint for small team |
| **Chosen: Custom first** | Prototype fast, adapt later | Will need GAS adapter if multiplayer added |

> [!contradiction] Multiplayer Risk
> If Persival adds multiplayer, the custom interface will need to be retrofitted with prediction and replication. GAS handles this natively. Flag this as a review trigger for the ADR.

### 2. Ability activation: direct call vs. queued
Current decision: **direct call** via `UAbilityManagerComponent::ActivateAbility()`. No queue. Abilities cancel each other. This fits single-player stealth; revisit for combo systems.

### 3. Cooldown ownership
Cooldown is owned by the **ability itself** (stored in the `IAbilitySystem` implementer), not by the manager. Manager queries `IsOnCooldown()` before activating.

---

## Review Trigger

Revisit this ADR if:
- Multiplayer / co-op is added → switch to GAS
- Ability queuing or combos are needed → add queue layer to `UAbilityManagerComponent`
- More than 6 abilities exist → evaluate if custom system scales

---

## Related
- [[Ability System Interface]] — C++ interface definition (virtual functions)
- [[Disguise Steal]] — first ability implementing this contract
- [[decisions/_index]] — promote to formal ADR when accepted
- [[meta/logbook]] — decision history

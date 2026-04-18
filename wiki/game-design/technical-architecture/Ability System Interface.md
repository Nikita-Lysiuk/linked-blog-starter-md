---
type: concept
title: IPR_AIAbilityTarget — Ability System Interface
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - technical-architecture
  - interface
  - persival
  - cpp
  - ability-system
status: implemented
complexity: intermediate
domain: software-architecture
related:
  - '[[IPR_AIAbilityTarget]]'
  - '[[APR_BaseAI]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[Disguise Steal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
  - '[[game-design/technical-architecture/_index]]'
---

# IPR_AIAbilityTarget — Ability System Interface

**Project:** Persival
**Language:** C++ (Unreal Engine 5.6)
**Status:** Implemented — lives in `Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h`

> [!correction] Previous version of this note described a fictional `IAbilitySystem` interface with `ExecuteAbility`, `CancelAbility`, and `UAbilityManagerComponent`. **None of that exists in the codebase.** The real ability interface is `IPR_AIAbilityTarget` — documented here.

---

## Intent

Define the **sole interaction boundary** between player abilities (P1 FaceSteal, P2 MindCopy) and AI characters. Ability code targets this interface exclusively — no concrete class casts, ever. This enforces the core design principle:

> "P1 and P2 abilities interact with AI exclusively through `IPR_AIAbilityTarget` — never by casting to a concrete class." — CLAUDE.md

---

## Interface Definition

```cpp
// Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h

UINTERFACE(MinimalAPI, Blueprintable)
class UPR_AIAbilityTarget : public UInterface { GENERATED_BODY() };

class IPR_AIAbilityTarget
{
    GENERATED_BODY()
public:
    // Returns false for mechanical enemies (EnemyConfig->bCanBeTargetedByAbility == false)
    UFUNCTION(BlueprintNativeEvent)
    bool CanBeTargetedByAbility() const;

    // P2 entry point — gives access to TakeSnapshot / Paste API
    UFUNCTION(BlueprintNativeEvent)
    UPR_AIMemoryComponent* GetMemoryComponent() const;

    // State query — consciousness state without casting to APR_BaseAI
    UFUNCTION(BlueprintNativeEvent)
    EAIConsciousnessState GetCurrentConsciousnessState() const;

    // Planned (PR_AIAbilityTarget.h:25–27): StealAppearance() for P1 full activation
};
```

---

## APR_BaseAI Implementation

| Method | Line | Implementation |
|--------|------|----------------|
| `CanBeTargetedByAbility_Implementation()` | `PR_BaseAI.h:85` | `return EnemyConfig->bCanBeTargetedByAbility` |
| `GetMemoryComponent_Implementation()` | `PR_BaseAI.h:90` | `return MemoryComponent` |
| `GetCurrentConsciousnessState_Implementation()` | `PR_BaseAI.h:95` | Delegates to `MemoryComponent->GetConsciousnessState()` |

---

## How Each Ability Uses the Interface

### P1 FaceSteal (Disguise Steal)
```
CanBeTargetedByAbility() → gate check
GetCurrentConsciousnessState() → verify not already Frozen/Replaced
[Then calls APR_BaseAI methods directly to add tags + set Frozen]
// StealAppearance() planned to unify this
```

### P2 MindCopy
```
CanBeTargetedByAbility() → gate check
GetMemoryComponent() → TakeSnapshot()         // Copy
GetMemoryComponent() → (via Controller)       // Paste A/B via APR_AIController
```

---

## Design Notes

> [!key-insight] Why Not GAS?
> Unreal's Gameplay Ability System was considered and rejected for now. See [[AI System Architecture]] for the full ADR. Short answer: GAS footprint is too large for a 3-person team at prototype stage; this interface + BT-driven state achieves the same result with full control.

> [!gap] StealAppearance() planned
> `PR_AIAbilityTarget.h:25–27` has comments marking `StealAppearance()` as a planned addition for P1's full activation path. Once P1's ability component is written, this method should be promoted to the interface contract.

---

## Related
- [[IPR_AIAbilityTarget]] — entity page with full class listing
- [[APR_BaseAI]] — sole current implementor
- [[UPR_AIMemoryComponent]] — exposed via `GetMemoryComponent()`
- [[Disguise Steal]], [[Mind Copy]] — ability pages using this interface
- [[AI System Architecture]] — why this pattern was chosen over GAS / custom manager

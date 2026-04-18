---
type: class
title: IPR_AIAbilityTarget
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - interface
  - cpp
  - persival
  - ability-system
  - ai
module: Interfaces
header: Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h
related:
  - '[[APR_BaseAI]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[Disguise Steal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
---

# IPR_AIAbilityTarget

**Header:** `Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h`
**Module:** Interfaces

The **sole interaction boundary** between player abilities (P1 FaceSteal, P2 MindCopy) and AI characters. All ability code targets this interface — no concrete class casts, ever.

---

## Contract

```cpp
UINTERFACE(MinimalAPI, Blueprintable)
class UPR_AIAbilityTarget : public UInterface { GENERATED_BODY() };

class IPR_AIAbilityTarget
{
    GENERATED_BODY()
public:
    // Returns false for mechanical enemies (EnemyConfig->bCanBeTargetedByAbility == false)
    UFUNCTION(BlueprintNativeEvent)
    bool CanBeTargetedByAbility() const;

    // P2 entry point — gives access to snapshot API
    UFUNCTION(BlueprintNativeEvent)
    UPR_AIMemoryComponent* GetMemoryComponent() const;

    // Query current consciousness state without casting
    UFUNCTION(BlueprintNativeEvent)
    EAIConsciousnessState GetCurrentConsciousnessState() const;
};
```

---

## APR_BaseAI Implementation

| Method | Line | Implementation |
|--------|------|----------------|
| `CanBeTargetedByAbility_Implementation()` | `PR_BaseAI.h:85` | `return EnemyConfig->bCanBeTargetedByAbility` |
| `GetMemoryComponent_Implementation()` | `PR_BaseAI.h:90` | `return MemoryComponent` |
| `GetCurrentConsciousnessState_Implementation()` | `PR_BaseAI.h:95` | Delegates to `MemoryComponent` |

---

## Design Rationale

> CLAUDE.md: "P1 and P2 abilities interact with AI exclusively through `IPR_AIAbilityTarget` — never by casting to a concrete class."

- A new enemy subclass needs only implement this interface to become ability-targetable.
- Mechanical enemies return `CanBeTargetedByAbility() = false` — immune with zero extra logic in ability code.
- `GetMemoryComponent()` gives P2 direct access to the snapshot API without knowing the NPC type.
- Planned: `StealAppearance()` method for P1's full activation — noted in `PR_AIAbilityTarget.h:25–27`.

---

## Relationships

- **Implemented by:** [[APR_BaseAI]]
- **Used by:** [[Disguise Steal]] (P1), [[Mind Copy]] (P2)
- **Exposes:** [[UPR_AIMemoryComponent]] to callers
- **Architecture:** [[AI System Architecture]]

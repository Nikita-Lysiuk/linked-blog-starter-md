---
type: class
title: IPR_AIAbilityTarget
created: '2026-04-18'
updated: '2026-04-27'
tags:
  - interface
  - cpp
  - persival
  - ability-system
  - ai
status: implemented
module: Interfaces
header: Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h
related:
  - '[[APR_BaseAI]]'
  - '[[UPR_AIMemoryComponent]]'
  - '[[FaceSteal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
---

# IPR_AIAbilityTarget

**Header:** `Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h`
**Module:** Interfaces
**Status:** Implemented on `APR_BaseAI`

The **query and access boundary** between player ability components and AI characters. All ability code reaches into AI exclusively through this interface — no concrete class casts from ability code, ever.

---

## Contract

```cpp
class IPR_AIAbilityTarget
{
    GENERATED_BODY()
public:
    // Returns false for mechanical enemies (EnemyConfig->bCanBeTargetedByAbility == false)
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    bool CanBeTargetedByAbility() const;

    // Gives ability components access to the snapshot and state API
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    UPR_AIMemoryComponent* GetMemoryComponent() const;

    // Called by any ability on deactivation — triggers 2s disorientation window
    // C++: sets bIsDisoriented BB key. BP override: plays stagger animation.
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    void BeginDisorientation();

    // Convenience state query without going through the component
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    EAIConsciousnessState GetCurrentConsciousnessState() const;
};
```

---

## APR_BaseAI Implementation

| Method | Implementation |
|--------|----------------|
| `CanBeTargetedByAbility_Implementation()` | `return EnemyConfig->bCanBeTargetedByAbility` |
| `GetMemoryComponent_Implementation()` | `return MemoryComponent` |
| `BeginDisorientation_Implementation()` | Sets `bIsDisoriented = true` on Blackboard. `BTTask_PR_Disorient` handles the 2s wake-up. BP subclass overrides to add animation. |
| `GetCurrentConsciousnessState_Implementation()` | Delegates to `MemoryComponent->GetConsciousnessState()` |

---

## Design Rationale

The interface is a **query/access surface only** — it exposes what's needed to reach into an AI, but defines no ability-specific behaviour. This means:

- A new ability component needs only `IPR_PlayerAbility` — it uses this interface to get what it needs and orchestrates the rest itself.
- Adding a new ability requires zero changes to `IPR_AIAbilityTarget`.
- `BeginDisorientation()` is on the interface because it is a general AI reaction called by any ability (not specific to FaceSteal or Telepathy).

---

## Relationships

- **Implemented by:** [[APR_BaseAI]]
- **Used by:** [[FaceSteal]] (P1 — `UPR_FaceStealComponent`), [[Mind Copy]] (P2 — pending)
- **Exposes:** [[UPR_AIMemoryComponent]] to callers
- **Architecture:** [[AI System Architecture]], [[Ability System Interface]]

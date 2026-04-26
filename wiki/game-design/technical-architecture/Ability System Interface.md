---
type: concept
title: IPR_AIAbilityTarget — Ability System Interface
created: '2026-04-18'
updated: '2026-04-27'
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
  - '[[FaceSteal]]'
  - '[[Mind Copy]]'
  - '[[AI System Architecture]]'
  - '[[game-design/technical-architecture/_index]]'
---

# IPR_AIAbilityTarget — Ability System Interface

**Project:** Persival
**Language:** C++ (Unreal Engine 5.6)
**Status:** Implemented — `Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h`

---

## Intent

Define the **query and access boundary** between player ability components and AI characters. Ability code uses this interface to ask two questions: *can I target this AI?* and *give me access to its internal components*. The ability component then orchestrates everything itself — the interface never knows what the ability does.

This enforces the Open/Closed Principle: adding a new ability requires only a new component implementing `IPR_PlayerAbility`, not a change to `IPR_AIAbilityTarget`.

---

## Interface Definition

```cpp
// Source/Nocturne/Public/Interfaces/PR_AIAbilityTarget.h

class IPR_AIAbilityTarget
{
    GENERATED_BODY()
public:
    // Gate check — false for mechanical enemies (EnemyConfig->bCanBeTargetedByAbility)
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    bool CanBeTargetedByAbility() const;

    // Access — gives ability components the snapshot API without knowing NPC type
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    UPR_AIMemoryComponent* GetMemoryComponent() const;

    // Trigger disorientation — called by any ability on deactivation/expiry
    // BlueprintNativeEvent: BP plays animation, C++ sets bIsDisoriented BB key
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    void BeginDisorientation();

    // State query shortcut — avoids component access for simple checks
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "AI|Abilities")
    EAIConsciousnessState GetCurrentConsciousnessState() const;
};
```

---

## APR_BaseAI Implementation

| Method | Implementation |
|--------|---------------|
| `CanBeTargetedByAbility_Implementation()` | `return EnemyConfig->bCanBeTargetedByAbility` |
| `GetMemoryComponent_Implementation()` | `return MemoryComponent` |
| `BeginDisorientation_Implementation()` | Sets `bIsDisoriented = true` on Blackboard — `BTTask_PR_Disorient` handles the 2s wake-up window. BP override plays stagger animation. |
| `GetCurrentConsciousnessState_Implementation()` | Delegates to `MemoryComponent->GetConsciousnessState()` |

---

## How Each Ability Uses the Interface

### P1 FaceSteal (`UPR_FaceStealComponent`)
```
CanBeTargetedByAbility()      → gate check (false = immunity snap)
GetCurrentConsciousnessState() → verify not already Frozen/Replaced
GetMemoryComponent()           → SetConsciousnessState(Frozen)
Cast to APR_BaseCharacter      → GetRuntimeTags() → copy to P1
Cast to ACharacter             → GetMesh() → store + swap
[On deactivate]
BeginDisorientation()          → triggers 2s wake-up, AlertLevel stays 0
```

The ability component orchestrates directly. The interface is a door opener — it never knows what FaceSteal does with what's behind the door.

### P2 Telepathy (`UPR_TelepathyComponent` — pending PR-89/90/91)
```
CanBeTargetedByAbility()  → gate check
GetMemoryComponent()      → TakeSnapshot(BB)           // Copy
GetMemoryComponent()      → (snapshot transfer)        // Paste A/B
[On Paste expiry]
BeginDisorientation()     → same wake-up path as P1
```

---

## Design Notes

> [!key-insight] Why Not GAS?
> Unreal's Gameplay Ability System was considered and rejected for now. See [[AI System Architecture]] for the full ADR. Short answer: GAS footprint (ASC on every NPC, Attribute Sets, Gameplay Effects) is too large for a 3-person team at prototype stage.

> [!key-insight] Why No Ability-Specific Methods?
> Previous design considered adding `StealAppearance()` to this interface. Rejected — it couples the interface to one specific ability, violating OCP. Every new ability would require modifying the interface. The current design keeps the interface as pure query/access; each ability component decides what to do with the access it gets.

---

## Related
- [[IPR_AIAbilityTarget]] — entity page with full class listing
- [[APR_BaseAI]] — sole current implementor
- [[UPR_AIMemoryComponent]] — exposed via `GetMemoryComponent()`
- [[FaceSteal]], [[Mind Copy]] — ability pages using this interface
- [[AI System Architecture]] — why this pattern was chosen over GAS

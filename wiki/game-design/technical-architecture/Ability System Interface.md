---
type: concept
title: "Ability System Interface (IAbilitySystem)"
created: 2026-04-18
updated: 2026-04-18
tags:
  - technical-architecture
  - pattern
  - persival
  - cpp
  - ability-system
status: developing
complexity: intermediate
domain: software-architecture
related:
  - "[[Universal Ability System Interface]]"
  - "[[Disguise Steal]]"
  - "[[Observer]]"
  - "[[game-design/technical-architecture/_index]]"
sources: []
---

# Ability System Interface — `IAbilitySystem`

**Project:** Persival
**Language:** C++ (Unreal Engine 5)
**Pattern:** Interface (pure virtual) — see [[Observer]] for event notification pattern used alongside this

---

## Intent

Define a minimal, enforced lifecycle contract for all hero abilities in Persival. Any `UObject` (or `AActor` subclass) that implements `IAbilitySystem` can be activated, cancelled, queried for cooldown, and slotted into `UAbilityManagerComponent` without the manager knowing the concrete ability type.

---

## Interface Definition

```cpp
// AbilitySystem.h
#pragma once
#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "AbilitySystem.generated.h"

UINTERFACE(MinimalAPI, Blueprintable)
class UAbilitySystem : public UInterface
{
    GENERATED_BODY()
};

/**
 * IAbilitySystem
 * Minimal lifecycle contract for all hero abilities in Persival.
 * Implement on any UObject or AActor subclass that represents an ability.
 */
class PERSIVAL_API IAbilitySystem
{
    GENERATED_BODY()

public:
    /**
     * ExecuteAbility
     * Called when the player activates this ability.
     * @param Instigator — The actor activating the ability (typically APlayerCharacter).
     *                     Must not be null. Implementer casts to expected type.
     */
    UFUNCTION(BlueprintNativeEvent, Category = "Ability")
    void ExecuteAbility(AActor* Instigator);
    virtual void ExecuteAbility_Implementation(AActor* Instigator) = 0;

    /**
     * CancelAbility
     * Called to interrupt an active ability before it expires naturally.
     * Must restore all state modified by ExecuteAbility.
     * Safe to call when ability is not active (no-op).
     */
    UFUNCTION(BlueprintNativeEvent, Category = "Ability")
    void CancelAbility();
    virtual void CancelAbility_Implementation() = 0;

    /**
     * GetCooldownRemaining
     * Returns seconds remaining on cooldown. Returns 0.0f if ability is ready.
     * Called by UAbilityManagerComponent before activating.
     */
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "Ability")
    float GetCooldownRemaining() const;
    virtual float GetCooldownRemaining_Implementation() const = 0;

    /**
     * IsOnCooldown
     * Convenience wrapper — returns true if GetCooldownRemaining() > 0.
     * Can be overridden if the implementer has a cheaper check.
     */
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "Ability")
    bool IsOnCooldown() const;
    virtual bool IsOnCooldown_Implementation() const
    {
        return GetCooldownRemaining() > 0.0f;
    }
};
```

---

## Ability Manager Component

```cpp
// AbilityManagerComponent.h
#pragma once
#include "AbilitySystem.h"
#include "Components/ActorComponent.h"
#include "AbilityManagerComponent.generated.h"

UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class PERSIVAL_API UAbilityManagerComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    /** Grant an ability to this component. */
    UFUNCTION(BlueprintCallable)
    void GrantAbility(TScriptInterface<IAbilitySystem> Ability);

    /**
     * Attempt to activate the ability in slot SlotIndex.
     * No-ops if slot is empty, ability is on cooldown, or Instigator is null.
     */
    UFUNCTION(BlueprintCallable)
    void ActivateAbility(int32 SlotIndex, AActor* Instigator);

    /** Cancel all currently active abilities. */
    UFUNCTION(BlueprintCallable)
    void CancelAllAbilities();

private:
    UPROPERTY()
    TArray<TScriptInterface<IAbilitySystem>> Abilities;
};
```

```cpp
// AbilityManagerComponent.cpp
#include "AbilityManagerComponent.h"

void UAbilityManagerComponent::GrantAbility(TScriptInterface<IAbilitySystem> Ability)
{
    if (Ability.GetObject())
        Abilities.Add(Ability);
}

void UAbilityManagerComponent::ActivateAbility(int32 SlotIndex, AActor* Instigator)
{
    if (!Abilities.IsValidIndex(SlotIndex) || !Instigator)
        return;

    auto& Ability = Abilities[SlotIndex];
    if (IAbilitySystem::Execute_IsOnCooldown(Ability.GetObject()))
        return;

    IAbilitySystem::Execute_ExecuteAbility(Ability.GetObject(), Instigator);
}

void UAbilityManagerComponent::CancelAllAbilities()
{
    for (auto& Ability : Abilities)
    {
        if (Ability.GetObject())
            IAbilitySystem::Execute_CancelAbility(Ability.GetObject());
    }
}
```

---

## Disguise Steal Implementation Skeleton

```cpp
// DisguiseAbility.h
#pragma once
#include "AbilitySystem.h"
#include "UObject/Object.h"
#include "DisguiseAbility.generated.h"

UCLASS()
class PERSIVAL_API UDisguiseAbility : public UObject, public IAbilitySystem
{
    GENERATED_BODY()

public:
    // Configurable in Blueprint / DataAsset
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Disguise")
    float DisguiseDuration = 60.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Disguise")
    float CooldownDuration = 30.0f;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Disguise")
    float DisguiseRange = 200.0f;

    // IAbilitySystem
    virtual void ExecuteAbility_Implementation(AActor* Instigator) override;
    virtual void CancelAbility_Implementation() override;
    virtual float GetCooldownRemaining_Implementation() const override;

private:
    bool bIsActive = false;
    float CooldownRemaining = 0.0f;
    float ActiveTimeRemaining = 0.0f;

    // Cached references for cleanup
    UPROPERTY()
    USkeletalMeshComponent* PlayerMesh = nullptr;

    UPROPERTY()
    USkeletalMesh* OriginalMesh = nullptr;

    TArray<UMaterialInterface*> OriginalMaterials;

    void StealGuardAppearance(AActor* Guard, USkeletalMeshComponent* PlayerMeshComp);
    void RestorePlayerAppearance();
    void StartCooldown();
};
```

```cpp
// DisguiseAbility.cpp — key methods
void UDisguiseAbility::ExecuteAbility_Implementation(AActor* Instigator)
{
    // 1. Find nearest valid guard within DisguiseRange
    // 2. Get guard's SkeletalMeshComponent
    // 3. Copy mesh + materials to player
    // 4. Notify AIPerceptionSystem of faction override
    // 5. Start duration timer → CancelAbility on expire
    bIsActive = true;
    ActiveTimeRemaining = DisguiseDuration;
}

void UDisguiseAbility::CancelAbility_Implementation()
{
    if (!bIsActive) return;
    RestorePlayerAppearance();
    bIsActive = false;
    StartCooldown();
}

float UDisguiseAbility::GetCooldownRemaining_Implementation() const
{
    return CooldownRemaining;
}
```

---

## Design Notes

> [!key-insight] Interface over Inheritance
> Using a UE5 interface (`UINTERFACE`) instead of a base class allows abilities to be implemented by both `UObject`-derived objects (data-only abilities) and `AActor`-derived objects (world-present abilities) without forcing a shared parent in the actor hierarchy.

> [!gap] No Ability Interruption Priority
> Current design has no priority system — abilities simply cancel each other. If two abilities activate simultaneously (e.g., an environment trigger and player input), the last caller wins. Define interrupt priority rules before shipping.

---

## Related
- [[Universal Ability System Interface]] — architecture overview and ADR
- [[Disguise Steal]] — first concrete implementation
- [[Observer]] — pattern used to notify AI perception on state change
- [[game-design/technical-architecture/_index]] — architecture index

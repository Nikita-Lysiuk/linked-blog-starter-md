---
type: class
title: APR_BasePlayer
created: '2026-04-18'
updated: '2026-04-27'
tags:
  - class
  - player
  - cpp
  - persival
status: implemented
module: Core
header: Source/Nocturne/Public/Core/PR_BasePlayer.h
related:
  - '[[AI System Architecture]]'
  - '[[FaceSteal]]'
  - '[[Mind Copy]]'
---

# APR_BasePlayer

**Header:** `Source/Nocturne/Public/Core/PR_BasePlayer.h`
**Module:** Core
**Inheritance:** `APR_BaseCharacter` → `ATurnInPlaceCharacter` (plugin) | implements `IPR_InputProvider`

Abstract base for both player characters (P1 Face-Stealer, P2 Telepath). Owns Enhanced Input wiring, sprint, grapple movement, and the stealth light-zone tracker. Each player's ability is a C++ component (`UPR_FaceStealComponent` / `UPR_TelepathyComponent`) assigned to `AbilityComponent` in their Blueprint subclass.

---

## Class Declaration

```cpp
UCLASS(Abstract, Blueprintable)
class APR_BasePlayer : public APR_BaseCharacter, public IPR_InputProvider
```

---

## Key Properties

### Input
| Property | Line | Notes |
|----------|------|-------|
| `MoveInputAction` | 28 | `UInputAction*` |
| `LookInputAction` | 31 | `UInputAction*` |
| `JumpInputAction` | 34 | `UInputAction*` |
| `CrouchInputAction` | 37 | `UInputAction*` |
| `SprintInputAction` | 40 | `UInputAction*` |
| `DefaultMappingContext` | 45 | `UInputMappingContext*` |
| `LookSensitivityX/Y` | 49–53 | float — per-player sensitivity |

### Components
| Property | Type | Line | Notes |
|----------|------|------|-------|
| `GrappleMovementComponent` | `UPR_GrappleMovementComponent*` | 93 | P2 vertical traversal |
| `StealthComponent` | `UPR_StealthComponent*` | 97 | Light zone overlap tracker |
| `AbilityComponent` | `TScriptInterface<IPR_PlayerAbility>` | ~100 | `EditDefaultsOnly` — set to `UPR_FaceStealComponent` in `BP_P1`, `UPR_TelepathyComponent` in `BP_P2` |

---

## Key Methods

| Method | Line | Notes |
|--------|------|-------|
| `IsSprinting()` | 56 | Returns `bIsSprinting` |
| `GetGrappleMovement()` | 59 | Returns grapple component |
| `GetStealthComponent()` | 62 | Returns stealth component |
| `BP_OnMoveInput()` | 81 | `BlueprintImplementableEvent` — move input forwarded to Blueprint |
| `BP_OnSprintChanged()` | 85 | `BlueprintImplementableEvent` — sprint state change |

---

## Ability System

`AbilityComponent` (`TScriptInterface<IPR_PlayerAbility>`) is the single ability slot per player. Each ability is a `UActorComponent` implementing `IPR_PlayerAbility`:

- `BP_P1_FaceStealer` assigns `UPR_FaceStealComponent` (PR-87 — in progress)
- `BP_P2_Telepath` assigns `UPR_TelepathyComponent` (PR-89/90/91 — pending)

Input bindings in `APR_BasePlayer` call `AbilityComponent->StartTargeting()`, `Activate()`, `Deactivate()` — the base player knows nothing about which specific ability is assigned.

`RuntimeTags` (for P1's stolen identity tags) is inherited from `APR_BaseCharacter`.

---

## Stealth Component

`UPR_StealthComponent` tracks overlapping `APR_LightZone` volumes and exposes an aggregate light multiplier used by `BTService_PR_UpdateAlertFromSight` to scale sight detection speed.

---

## Movement Speeds

| Mode | Speed |
|------|-------|
| Walk | 600 cm/s |
| Crouch | 300 cm/s |
| Sprint | 1200 cm/s |

Sprint increases AI perception significantly.

---

## Inheritance Chain

```
ATurnInPlaceCharacter (Plugin)
  └── APR_BaseCharacter  (RuntimeTags, tag CRUD)
        └── APR_BasePlayer  (+IPR_InputProvider, +AbilityComponent)
              ├── BP_P1_FaceStealer  (UPR_FaceStealComponent)
              └── BP_P2_Telepath     (UPR_TelepathyComponent)
```

---

## Relationships

| Relationship | Target |
|-------------|--------|
| P1 ability | [[FaceSteal]] — `UPR_FaceStealComponent` on `AbilityComponent` |
| P2 ability | [[Mind Copy]] — `UPR_TelepathyComponent` on `AbilityComponent` (pending) |
| Architecture | [[AI System Architecture]] |

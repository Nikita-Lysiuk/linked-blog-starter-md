---
type: class
title: APR_BasePlayer
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - class
  - player
  - cpp
  - persival
module: Core
header: Source/Nocturne/Public/Core/PR_BasePlayer.h
related:
  - '[[AI System Architecture]]'
  - '[[Disguise Steal]]'
  - '[[Mind Copy]]'
---

# APR_BasePlayer

**Header:** `Source/Nocturne/Public/Core/PR_BasePlayer.h`
**Module:** Core
**Inheritance:** `APR_BaseCharacter` → `ATurnInPlaceCharacter` (plugin) | implements `IPR_InputProvider`

Abstract base for both player characters (P1 Face-Stealer, P2 Telepath). Owns Enhanced Input wiring, sprint, grapple movement, and the stealth light-zone tracker. Concrete player abilities live in Blueprint subclasses.

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
| Property | Line | Notes |
|----------|------|-------|
| `GrappleMovementComponent` | 93 | `UPR_GrappleMovementComponent*` — P2 vertical traversal |
| `StealthComponent` | 97 | `UPR_StealthComponent*` — light zone overlap tracker |

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

## Stealth Component

`UPR_StealthComponent` tracks overlapping `APR_LightZone` volumes and exposes an **aggregate light multiplier** used by AI perception to scale sight detection. Darker zones reduce the effective sight radius on the per-NPC level.

---

## Grapple Hook (P2)

`UPR_GrappleHookComponent` provides cone-based anchor targeting:
- Radius: **1500 cm**
- Dot threshold: **0.4** (±66° cone)
- Critical for P2's vertical traversal between floors

---

## Movement Speeds (`UPR_CharacterMovement`)

| Mode | Speed |
|------|-------|
| Walk | 600 cm/s |
| Crouch | 300 cm/s |
| Sprint | 1200 cm/s |

Sprint is loud and highly visible — increases AI perception significantly.

---

## Inheritance Chain

```
ATurnInPlaceCharacter (Plugin: TurnInPlace-release/)
  └── APR_BaseCharacter
        └── APR_BasePlayer  (+IPR_InputProvider)
              ├── BP_P1_FaceStealer  (Blueprint — FaceSteal ability)
              └── BP_P2_Telepath     (Blueprint — MindCopy ability)
```

---

## Relationships

| Relationship | Target |
|-------------|--------|
| P1 ability | [[Disguise Steal]] — tags copied onto this actor's container |
| P2 ability | [[Mind Copy]] — activates Copy/Paste via controller calls |
| Architecture | [[AI System Architecture]] |

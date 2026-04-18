---
type: class
title: UPR_EnemyConfig
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - class
  - data-asset
  - ai
  - cpp
  - persival
module: AI
header: Source/Nocturne/Public/AI/PR_EnemyConfig.h
related:
  - '[[APR_BaseAI]]'
  - '[[APR_AIController]]'
  - '[[Disguise Steal]]'
  - '[[AI System Architecture]]'
---

# UPR_EnemyConfig

**Header:** `Source/Nocturne/Public/AI/PR_EnemyConfig.h`
**Module:** AI
**Base:** `UDataAsset`
**Asset prefix:** `DA_` (e.g. `DA_Guard`, `DA_Officer`, `DA_Mech`)
**Asset path:** `/Game/AI/Configs/`

The single source of truth for every enemy's parameters. Nothing gameplay-relevant is hardcoded in C++ — all tuning lives here. One DataAsset per enemy archetype; Blueprint subclasses create instances.

---

## Configuration Domains

### Identity
| Property | Purpose |
|----------|---------|
| `DefaultTags` | `FGameplayTagContainer` — faction + type tags copied to `APR_BaseAI::RuntimeTags` at `BeginPlay` |
| `bCanBeTargetedByAbility` | `false` on mechanical enemies — makes them immune to P1 FaceSteal and P2 MindCopy |

### Movement
| Property | Notes |
|----------|-------|
| Walk / Crouch / Sprint speeds | cm/s, fed to `UPR_CharacterMovement` |
| Patrol config | `EPatrolMode`, wait times at points |

### Perception
| Property | Notes |
|----------|-------|
| Sight radius, angle, range | Fed to `UPR_AIPerceptionComponent` |
| Hearing radius | Threshold distances per noise level |
| `AlertDecay` rate | How fast `AlertLevel` falls when no stimuli |

### Suspicion / Ability
| Property | Line | Notes |
|----------|------|-------|
| `InvestigateWaitTime` | — | Seconds NPC waits at `LastKnownLocation` |
| `SearchRadius` | — | Random search area radius |
| `SearchDuration` | — | Max seconds spent searching |
| `DisorientDuration` | — | Freeze time at start of P2 Paste A/B |
| `MindReplaceDuration` | — | How long Paste B replacement lasts |
| `FaceStealRange` | 193–196 | Max distance P1 can maintain stolen appearance (cm) |

### Timing / Dialog
| Property | Notes |
|----------|-------|
| `InvestigateAnimDuration` | Duration of notice reaction animation |
| `DialogConfig` | `UPR_DialogConfig*` — ambient dialogue lines |

---

## Usage

```cpp
// APR_BaseAI reads config at BeginPlay
EnemyConfig->DefaultTags       // → InitializeRuntimeTags()
EnemyConfig->bCanBeTargetedByAbility  // → CanBeTargetedByAbility_Implementation()
EnemyConfig->FaceStealRange    // → P1 range check
EnemyConfig->DisorientDuration // → APR_AIController::ApplyPasteA/B
EnemyConfig->MindReplaceDuration // → ApplyPasteB MindReplaceEndTime calc
```

**Adding a new enemy archetype:**
1. Create `DA_<EnemyName>` in `/Game/AI/Configs/`
2. Set `bCanBeTargetedByAbility = false` for mechanical enemies
3. Create a thin Blueprint subclass of `APR_BaseAI` referencing the new DataAsset
4. No C++ changes required

---

## Relationships

| Relationship | Target |
|-------------|--------|
| Owner | [[APR_BaseAI]] — reads at `BeginPlay` |
| Exposes immunity flag | [[IPR_AIAbilityTarget]]`::CanBeTargetedByAbility()` |
| FaceSteal config | [[Disguise Steal]] — `FaceStealRange` |
| Paste timing | [[Mind Copy]] — `DisorientDuration`, `MindReplaceDuration` |

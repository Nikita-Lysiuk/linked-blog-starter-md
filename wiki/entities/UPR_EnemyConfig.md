---
type: class
title: UPR_EnemyConfig
created: '2026-04-18'
updated: '2026-04-27'
tags:
  - class
  - data-asset
  - ai
  - cpp
  - persival
status: implemented
module: AI
header: Source/Nocturne/Public/AI/PR_EnemyConfig.h
related:
  - '[[APR_BaseAI]]'
  - '[[APR_AIController]]'
  - '[[FaceSteal]]'
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
| `DefaultTags` | `FGameplayTagContainer` — type tags (e.g. `AI.Type.Guard`) copied to `APR_BaseCharacter::RuntimeTags` at `BeginPlay` via `InitializeRuntimeTags()` |
| `bCanBeTargetedByAbility` | `false` on mechanical enemies — immune to P1 FaceSteal and P2 MindCopy |

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
| Property | Notes |
|----------|-------|
| `InvestigateWaitTime` | Seconds NPC waits at `LastKnownLocation` |
| `SearchRadius` | Random search area radius |
| `SearchDuration` | Max seconds spent searching |
| `DisorientDuration` | Duration of `BTTask_PR_Disorient` wake-up window after any ability deactivation |
| `MindReplaceDuration` | How long Paste B replacement lasts |
| `FaceStealRange` | Still present in config — may be repurposed for level-design annotations. **Not used for ability range check** — `UPR_FaceStealComponent::StealRadius` is the authoritative range for the ability. |
| `SuspiciousColleagueTime` | How long a frozen NPC goes unnoticed by nearby colleagues |

### Timing / Dialog
| Property | Notes |
|----------|-------|
| `InvestigateAnimDuration` | Duration of notice reaction animation |
| `DialogConfig` | `UPR_DialogConfig*` — ambient dialogue lines |

---

## Usage

```cpp
// APR_BaseAI reads config at BeginPlay
EnemyConfig->DefaultTags              // → InitializeRuntimeTags()
EnemyConfig->bCanBeTargetedByAbility  // → CanBeTargetedByAbility_Implementation()
EnemyConfig->DisorientDuration        // → BTTask_PR_Disorient duration
EnemyConfig->MindReplaceDuration      // → Paste B MindReplaceEndTime calc
EnemyConfig->SuspiciousColleagueTime  // → BTService_PR_CheckColleagues threshold
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
| Immunity flag | [[IPR_AIAbilityTarget]]`::CanBeTargetedByAbility()` |
| Paste timing | [[Mind Copy]] — `DisorientDuration`, `MindReplaceDuration` |

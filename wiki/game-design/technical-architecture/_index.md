---
type: domain
title: Technical Architecture Index
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - meta
  - technical-architecture
  - game-design
status: developing
page_count: 5
---

# Technical Architecture

> System diagrams, class hierarchies, data flows, and implementation notes for each mechanic.

## Architecture Documents

| Document | Status | Purpose |
|----------|--------|---------|
| [[AI System Architecture]] | implemented | Full AI subsystem map — class hierarchy, ability flows, design principles, replication notes |
| [[Consciousness State Machine]] | implemented | `EAIConsciousnessState` transitions — Frozen / Overwritten / Replaced / Normal |
| [[Ability System Interface]] | implemented | `IPR_AIAbilityTarget` — the P1/P2 → AI interaction boundary |
| [[Universal Ability System Interface]] | decided | ADR: why BT-driven state was chosen over custom ability manager / GAS |

## Mechanic Implementation Notes

| Mechanic | Impl Note |
|----------|-----------|
| [[Disguise Steal]] | [[AI System Architecture]] §P1 FaceSteal flow + [[Consciousness State Machine]] §Frozen |
| [[Mind Copy]] | [[AI System Architecture]] §P2 flows + [[Consciousness State Machine]] §Overwritten / Replaced |

## Key Source Paths

| Path | Contents |
|------|----------|
| `Source/Nocturne/Public/AI/` | `PR_BaseAI.h`, `PR_AIController.h`, `PR_EnemyConfig.h`, `PR_AITypes.h`, `PR_AIBlackboardKeys.h` |
| `Source/Nocturne/Public/AI/Components/` | `PR_AIMemoryComponent.h`, `PR_AIPerceptionComponent.h` |
| `Source/Nocturne/Public/AI/Services/` | DecayAlert, UpdateAlertFromSight, MindReplaceExpiry, ReadGlobalState, CheckColleagues |
| `Source/Nocturne/Public/AI/Tasks/` | HandleFrozen, HandleOverwritten, ClearAlert, GoToPatrolPoint, SearchArea, etc. |
| `Source/Nocturne/Public/Interfaces/` | `PR_AIAbilityTarget.h`, `PR_InputProvider.h` |
| `Source/Nocturne/Public/Core/` | `PR_BasePlayer.h`, `PR_BaseCharacter.h`, `PR_GameMode.h` |
| `/Game/AI/Configs/` | DataAssets: `DA_Guard`, `DA_Officer`, `DA_Mech` |
| `/Game/AI/BT/` | `BT_AIBase`, `BB_AIBase` |

## Related
- [[wiki/entities/_index]] — C++ class entity pages
- [[wiki/game-design/mechanics/_index]] — mechanics being implemented
- [[wiki/patterns/_index]] — design patterns applied (Observer, Memento)
- [[wiki/decisions/_index]] — ADRs

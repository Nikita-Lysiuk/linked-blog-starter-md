---
type: domain
title: Entities Index
created: '2026-04-18T00:00:00.000Z'
updated: '2026-04-18T00:00:00.000Z'
tags:
  - meta
  - entities
status: developing
page_count: 6
---

# Entities

> C++ classes, interfaces, and data assets that form the Nocturne codebase. For people/orgs/tools, type is `person`/`organization`/`tool`.

## C++ Classes — AI Subsystem

| Entity | Type | Header | Purpose |
|--------|------|--------|---------|
| [[APR_BaseAI]] | `class` | `Public/AI/PR_BaseAI.h` | Base pawn for all NPCs — Guards, Officers, Civilians |
| [[APR_AIController]] | `class` | `Public/AI/PR_AIController.h` | Perception, alert tracking, P2 ability entry points |
| [[UPR_AIMemoryComponent]] | `component` | `Public/AI/Components/PR_AIMemoryComponent.h` | Consciousness state + P2 snapshot API |
| [[UPR_EnemyConfig]] | `data-asset` | `Public/AI/PR_EnemyConfig.h` | All per-enemy tunable parameters |
| [[IPR_AIAbilityTarget]] | `interface` | `Public/Interfaces/PR_AIAbilityTarget.h` | P1/P2 ability interaction boundary |

## C++ Classes — Core

| Entity | Type | Header | Purpose |
|--------|------|--------|---------|
| [[APR_BasePlayer]] | `class` | `Public/Core/PR_BasePlayer.h` | Abstract player base — Enhanced Input, grapple, stealth |

## Related
- [[wiki/index]] — master catalog
- [[AI System Architecture]] — how these classes interconnect

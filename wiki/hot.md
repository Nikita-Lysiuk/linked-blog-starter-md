---
type: meta
title: Hot Cache
updated: '2026-04-18T00:00:00.000Z'
tags:
  - meta
  - hot-cache
---

# Recent Context

## Last Updated
2026-04-18. Full wiki sync against actual codebase. All fictional content corrected.

## Current Sprint
**Project Persival** — AI Sprint.

**Active tasks:**
- P1 FaceSteal: infrastructure complete (tags, Frozen state, IPR_AIAbilityTarget). **Next: write the player-facing activation component** that calls `AddGameplayTag` on P1 and `SetConsciousnessState(Frozen)` on target.
- P2 MindCopy: BT/BB/MemoryComponent fully implemented. **Next: write the player-facing Copy/Paste component.**
- Both abilities need a snapshot HUD indicator.

**Learning Campus** — [[Collisions]] theory page complete. Collision Sandbox project staged.

## Active Technical Roadblocks

1. **FaceSteal — Player activation component missing.** `StealAppearance()` noted as planned in `PR_AIAbilityTarget.h:25–27` but not yet promoted to interface. P1 ability component hasn't been written.

2. **P1+P2 interaction — Paste A on Frozen NPC.** If P2 Paste A restores `Normal` on a Frozen NPC, P1's disguise breaks silently. Edge case unresolved — needs cooperative design pass.

3. **MindCopy — Paste B on already-Replaced NPC.** State interaction with P1 simultaneous target undefined. Needs ADR.

## Key Architecture Facts (verified against code)
- AI perception uses **GameplayTags** for identity — not mesh or class type (`APR_BaseAI::RuntimeTags`, replicated, `PR_BaseAI.h:196`)
- P1 disguise = **tag injection + Frozen state** — no mesh swap, no `UDisguiseComponent`
- P2 MindCopy = **Blackboard snapshot** via `UPR_AIMemoryComponent` — server-only, not replicated
- Ability interface: **`IPR_AIAbilityTarget`** — 3 methods: `CanBeTargetedByAbility`, `GetMemoryComponent`, `GetCurrentConsciousnessState`
- No GAS, no `IAbilitySystem`, no `UAbilityManagerComponent` — those were discarded proposals
- Consciousness states: `Normal` / `Frozen` (P1) / `Overwritten` (P2 transition) / `Replaced` (P2 Paste B)

## Recent Changes (this session)
- **CORRECTED:** [[Disguise Steal]] — complete rewrite. Old version described mesh swap + `UDisguiseComponent` + `IAbilitySystem` (fictional). Now reflects actual tag-injection + Frozen state implementation.
- **CORRECTED:** [[Ability System Interface]] — rewritten to describe `IPR_AIAbilityTarget`, not `IAbilitySystem`.
- **CORRECTED:** [[Universal Ability System Interface]] — rewritten as ADR explaining why BT-driven state was chosen over the discarded custom ability manager proposal.
- **CREATED:** [[Mind Copy]] — full P2 ability mechanic page (Copy, Paste A, Paste B flows)
- **CREATED:** [[AI System Architecture]] — complete system map with class hierarchy, ability flows, design principles
- **CREATED:** [[Consciousness State Machine]] — state transition diagram and detail for all 4 states
- **CREATED:** [[IPR_AIAbilityTarget]], [[APR_BaseAI]], [[UPR_AIMemoryComponent]], [[APR_AIController]], [[UPR_EnemyConfig]], [[APR_BasePlayer]] entity pages
- **UPDATED:** `wiki/entities/_index.md`, `wiki/game-design/technical-architecture/_index.md`

## Active Threads
- Next: write P1 FaceSteal activation component
- Next: write P2 MindCopy activation component
- Next: define P1+P2 simultaneous-target interaction (ADR)
- Open: flesh out `wiki/game-design/loop-logic/` — core stealth loop not yet defined
- Open: Collision Sandbox Stage 1 (math setup)

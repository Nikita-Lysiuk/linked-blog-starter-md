---
type: meta
title: "Nocturne — Dashboard"
updated: 2026-04-27
tags: [meta, dashboard]
status: active
---

# 🌑 Nocturne — Project Dashboard

**Studio:** Persival &nbsp;·&nbsp; **Engine:** Unreal Engine 5.6 &nbsp;·&nbsp; **Sprint:** AI + Ability Sprint &nbsp;·&nbsp; **Updated:** 2026-04-27

---

## 🔥 Active Now

> [!danger]+ 🔥 Hot Context — Read This First
> Sprint context, current blockers, and what to pick up next.
> 
> **Active milestone:** `UPR_FaceStealComponent` — header done, implementation pending
> **P1 next:** Implement `.cpp` method bodies per locked design
> **P2 next:** Begin `UPR_TelepathyComponent` after FaceSteal ships
> 
> → [[hot|Open Hot Page →]]

---

## 📋 Sprint Board

> [!bug]+ 🔨 In Progress
> - [ ] `UPR_FaceStealComponent.cpp` — implement all 7 interface methods + timer bodies
> - [ ] `BTService_PR_UpdateAlertFromSight.cpp` — add `Ability.FaceSteal.Active` tag check to `TickNode`
> - [ ] Fix `BLueprintNativeEvent` typo in `PR_AIAbilityTarget.h` line 62
> - [ ] Remove stale `StealAppearance` / `PasteConsciousness` comments from `PR_AIAbilityTarget.h`

> [!success]+ ✅ Done This Sprint
> - [x] `IPR_PlayerAbility` — all 7 methods defined and reviewed
> - [x] `UPR_FaceStealComponent.h` — header complete (enum, class name, ClassGroup fixed)
> - [x] `DefaultGameplayTags.ini` — pruned to 5 canonical tags
> - [x] `APR_BaseCharacter` — `RuntimeTags` + 4 methods migrated from `APR_BaseAI`
> - [x] `BTService_PR_UpdateAlertFromSight.h` — node memory cleaned, tag moved to file scope
> - [x] Wiki — 9 pages updated, `Disguise Steal.md` deleted (superseded)

> [!warning]+ ⏭ Up Next
> - [ ] `UPR_TelepathyComponent.h` — design session + header
> - [ ] `BTService` — full sight alert math (distanceFactor × angleFactor × lightFactor)
> - [ ] AI Sprint review + demo prep

---

## 🎮 Mechanics

> [!tip]+ 👤 FaceSteal — P1 Core Ability
> Steal enemy mesh + GameplayTags. Social stealth via tag rank/role system.
> Two-phase: **E** → lock target (`Targeting`) · **LMB** → confirm (`Active`)
> States: `Inactive → Targeting → Active → GraceTimer`
> 
> → [[game-design/mechanics/FaceSteal|FaceSteal Design →]]

> [!tip]+ 🧠 Telepathy — P2 Core Ability
> Manipulate AI `ConsciousnessState`. States: `Normal → Frozen → Replaced`.
> Copy/Paste = full Blackboard state transfer. Replaced runs normal BT with expiry service.
> 
> → [[game-design/mechanics/Telepathy|Telepathy Design →]]

> [!tip]+ 🤖 AI Social Behavior
> GameplayTag-driven social restriction. Guards fooled by matching tags.
> Officers run secondary validation if `SuspicionLevel` is high.
> 
> → [[game-design/mechanics/AI_Social_Behavior|AI Social Behavior →]]

> [!tip]+ 📋 Mind Copy
> P2's Copy/Paste — snapshots and transfers the full Blackboard state to a Replaced AI.
> 
> → [[game-design/mechanics/Mind Copy|Mind Copy Design →]]

---

## 🏗️ Technical Architecture

> [!abstract]+ 🤖 AI System Architecture
> Perception pipeline, alert ramping, BT service layout, `UPR_AIMemoryComponent` rollback.
> 
> → [[game-design/technical-architecture/AI System Architecture|AI System Architecture →]]

> [!abstract]+ 🔌 Ability System Interface
> `IPR_PlayerAbility` (7 methods) + `IPR_AIAbilityTarget` (query/access only — SOLID).
> New abilities only implement `IPR_PlayerAbility`. AI interface never changes.
> 
> → [[game-design/technical-architecture/Ability System Interface|Ability System Interface →]]

> [!abstract]+ 🧠 Consciousness State Machine
> `Normal → Frozen → Replaced` transitions, BT branch consequences, expiry service design.
> 
> → [[game-design/technical-architecture/Consciousness State Machine|Consciousness State Machine →]]

> [!abstract]+ 🔗 Universal Ability System — ADR
> Architecture decision record for cross-cutting ability ↔ AI interaction pattern.
> 
> → [[game-design/technical-architecture/Universal Ability System Interface|ADR: Universal Ability System →]]

---

## 💻 Code Entities

| Entity | Role | Status |
|--------|------|--------|
| [[entities/APR_BaseAI\|APR_BaseAI]] | NPC pawn base · perception owner · BT host | Stable |
| [[entities/APR_BasePlayer\|APR_BasePlayer]] | Player pawn · Enhanced Input · ability host | Updated |
| [[entities/APR_AIController\|APR_AIController]] | BT runner · alert escalation | Stable |
| [[entities/IPR_AIAbilityTarget\|IPR_AIAbilityTarget]] | Query interface — ability ↔ AI bridge | ⚠ Typo on L62 |
| [[entities/UPR_AIMemoryComponent\|UPR_AIMemoryComponent]] | 10s Blackboard rollback (Memento pattern) | Stable |
| [[entities/UPR_EnemyConfig\|UPR_EnemyConfig]] | Per-NPC DataAsset config · ability immunity flags | Stable |

---

## 🗺️ Level Design

> [!note]+ 🏙️ Level 01 — Street to Mansion
> Tutorial level. Three zones: Street (stealth intro) → Clockmaker Apartment (co-op puzzle + lore) → Inner Courtyard (escalation). Two-tier split: P1 ground / P2 rooftops throughout.
> Status: `concept`
>
> → [[game-design/level-design/Level_01_Street_To_Mansion|Level 01 Design →]]
> → [[game-design/level-design/Puzzle_Clockmaker_Apartment|Clockmaker Apartment Puzzle →]]

---

## 🌍 World & Lore

> [!note]+ 📜 World Lore
> Dark industrial steampunk city. Dictator purged all gifted people 100 years ago — current generation has no memory they existed. Heroes are survivors' descendants working as thieves.
>
> → [[game-design/World_Lore|World Lore →]]

---

## 📐 Design Documents

> [!note]- 📖 GDD — Nocturne
> Full Game Design Document. Vision, protagonists, design pillars, core loop.
> 
> → [[game-design/GDD_Nocturne|Open GDD →]]

> [!note]- 📋 Ability Sprint Implementation Plan
> Agreed implementation order: `IPR_PlayerAbility → FaceSteal → Telepathy → AI polish`
> 
> → [[game-design/AbilitySprint_Implementation|Open Sprint Plan →]]

---

## 📝 Sessions

| Date | Topic |
|------|-------|
| [[sessions/2026-06-15_FaceSteal_VFX_UI_Niagara\|2026-06-15]] | FaceSteal VFX + UI + Niagara rope — outline material, HUD widget, C++ Niagara integration |
| [[sessions/2026-04-27_FaceStealComponent_Design\|2026-04-27]] | FaceStealComponent — full design grill session, header finalized |
| [[sessions/2026-04-26_FaceSteal_Sprint\|2026-04-26]] | FaceSteal Sprint kickoff, tag system redesign |

---

## 🗂️ Meta

> [!info] Quick Links
> [[meta/overview|🗺 Canvas Overview]] &nbsp;·&nbsp; [[meta/lint-report-2026-04-27|🔍 Lint Report (2026-04-27)]] &nbsp;·&nbsp; [[log|📜 Change Log]] &nbsp;·&nbsp; [[decisions/_index|📑 ADR Index]]

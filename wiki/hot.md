---
type: hot-cache
title: Current Sprint Context
updated: '2026-04-25'
tags:
  - hot
  - sprint
---

# Hot Cache — Ability Sprint

## Current Sprint: Ability Sprint (Sprint 4)

**Goal:** Connect P1 and P2 abilities to the AI system that was built in AI Sprint. Make the stealth loop actually playable.

## Priority Order
```
PR-88 → PR-92 → PR-87 → PR-89 → PR-90 → PR-91 → PR-94 → PR-93
```

## Do Next
**PR-88 — Wire `IPR_AIAbilityTarget` on `APR_BaseAI`**
- This unblocks everything else
- See [[AbilitySprint_Implementation]] for full requirements

## What Was Just Finished (AI Sprint)
- Full patrol system (all 4 modes, shared paths)
- Two-layer alert system (0→0.5 investigate, 0.5→1.0 failure)
- Consciousness states: Frozen, Overwritten, Replaced
- `ApplyPasteA()` / `ApplyPasteB()` on `APR_AIController`
- Colleague awareness (`BTService_PR_CheckColleagues`)
- Stealth light system (`APR_LightZone` + `UPR_StealthComponent`)
- GameplayTag set in `DefaultGameplayTags.ini`
- BT editor: services on root, Abnormal Consciousness branch, NoticeReaction

## Key Design Decisions Locked In
- Rope break → NPC Option A: AlertLevel stays 0, 1–2 sec disorientation (Act 1)
- Rope break on immune NPC → AlertLevel 0.5 (Act 2-3)
- FaceSteal re-use on same NPC: allowed (levels designed around it)
- Replaced state runs normal BT with donor's BB data (not a blocked state)
- Two game halves: stealth (fear/tension) + puzzle (curiosity/aha)
- Full design: [[GDD_Nocturne]]

## Jira Tickets
| Key | Summary | Status |
|-----|---------|--------|
| PR-88 | Wire IPR_AIAbilityTarget | To Do |
| PR-87 | UFaceStealComponent (P1) | To Do |
| PR-89 | P2 Telepathy — Copy | To Do |
| PR-90 | P2 Telepathy — Paste A | To Do |
| PR-91 | P2 Telepathy — Paste B | To Do |
| PR-92 | Apply AI.State.* tags | To Do |
| PR-93 | AI perception tuning | To Do |
| PR-94 | Place LightZone volumes | To Do |

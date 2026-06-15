---
type: level-design
title: Level 01 — Street to Mansion
created: '2026-06-15'
updated: '2026-06-15'
tags:
  - level-design
  - level-01
  - tutorial
  - act-1
status: concept
---

# Level 01 — Street to Mansion

## Overview

**Goal:** Reach the mansion entrance.
**Setting:** Night. City street outside mansion grounds → inner courtyard.
**Player count:** 2 (local split-screen, P1 + P2)

Players begin on a public street near the mansion perimeter. They must find a way through the perimeter wall, cross the inner courtyard, and reach the mansion entrance door — the level-end trigger.

---

## Macro Flow

```
[Street] → [Clockmaker Apartment] → [Tunnel] → [Vista] → [Inner Courtyard] → [Mansion Door]
```

**Pacing: Clench → Release → Clench**

| Zone | Tension | Purpose |
|---|---|---|
| Street | Medium — first guards | Teach stealth fundamentals, establish A/B split |
| Apartment | None — no guards | Decompression, co-op puzzle, lore delivery |
| Tunnel → Vista | None → payoff | Transition beat, mansion revealed for first time |
| Inner Courtyard | High — more guards | Apply everything learned, social stealth under pressure |

---

## The Two-Tier Split

The entire level runs on a **ground / upper** split that mirrors the players' asymmetry:

| Tier | Player | Why |
|---|---|---|
| Ground | P1 (Face-Stealer) | Face-steal requires close range — P1 must be at NPC level |
| Rooftops / upper floors | P2 (Telepath) | Grapple hook enables vertical movement |

**Design rule:** Grapple anchor points are designer-controlled. The perimeter wall has **no** grapple anchors. It is visually impassable (patrolled parapet, visible spikes) so players understand intuitively why they can't simply grapple over — they need the tunnel.

---

## Zone 1 — Street (Tutorial Stealth)

**What players learn:**
- Guards exist and patrol in patterns
- Light/shadow matters
- P2 can reach rooftops; P1 cannot — the split is established here before it matters

**NPC density:** Low. It's past curfew — citizens are indoors, only guards patrol. Players have room to breathe and try things.

**Curfew detail:** No civilians on street. This justifies sparse NPC count without breaking world logic.

**Entry hook:** Players receive a client briefing (note/radio) before or at level start:
> *"Old clockmaker's workshop, south wall, back alley door."*

This gives players a lead — they're not pixel-hunting for the entrance.

---

## Zone 2 — Clockmaker Apartment (Puzzle)

See [[Puzzle_Clockmaker_Apartment]] for full puzzle design.

**Function in level flow:**
- No enemies — safe decompression after street stealth
- Introduces the **dumbwaiter verb** (passing items between tiers) — will recur throughout the level
- Introduces **simultaneous activation** — core co-op mechanic
- Delivers the first lore beat: the Night of the Purge, the gifted family, why the tunnel exists

**The apartment is two-tiered in miniature.**
P1 enters through the back alley ground door. P2 grapples to the attic/roof window. Interior stairs are collapsed — they are immediately separated. Players rehearse the A-below / B-above dynamic in a safe, no-pressure environment before the courtyard demands it under guard scrutiny.

---

## Zone 3 — Vista Moment

Players exit the tunnel and emerge onto an elevated ledge or raised ground — the first unobstructed view of the mansion.

**Design intent:** This is the reward for solving the puzzle. The level's goal is revealed visually for the first time. Should feel monumental — the Spencer Mansion reveal. Slightly ominous, clearly beautiful, clearly dangerous.

Allow a beat here. No guards within earshot. Let players look.

---

## Zone 4 — Inner Courtyard (Escalation)

**What players apply:**
- Higher NPC density — stealth mechanics under real pressure
- Social stealth: P1 can Face-Steal a guard's appearance and walk past rank-restricted checkpoints
- P2 can Telepath guards from upper vantage points to clear paths for P1

**Goal:** Reach the mansion entrance door → level end.

**Guard layout:** TBD during blockout. At minimum: two patrol routes, one stationary post at the mansion entrance, one elevated guard P2 can access.

---

## Technical Notes

| Item | Detail |
|---|---|
| Map | `Lvl_00` (main level) or a dedicated `Lvl_01` — TBD |
| Streaming | Street / Apartment / Courtyard as separate streaming zones |
| Split-screen | Vertical split. HUD: minimal — no minimap in tutorial zone |
| Grapple anchors | Designer-placed. None on perimeter wall. Apartment roof has one (teaches grapple). Courtyard walls have 2–3. |
| Curfew VFX | Dark, fog, lamplight pools — amplify stealth readability |

---

## Status

`concept` — flow agreed, puzzle designed, courtyard layout TBD

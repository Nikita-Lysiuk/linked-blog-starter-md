---
type: puzzle-design
title: Puzzle — Clockmaker Apartment
created: '2026-06-15'
updated: '2026-06-15'
tags:
  - puzzle
  - level-01
  - co-op
  - lore
  - clockmaker
  - dumbwaiter
status: concept
---

# Puzzle — Clockmaker Apartment

## Summary

A three-beat co-op puzzle set inside a century-old clockmaker's workshop built into a tenement adjoining the mansion's perimeter wall. Solving it reveals a hidden tunnel — the clockmaker's escape route from the night of the Purge — which the players traverse in reverse as an infiltration route.

**Each beat teaches one co-op verb. No beat can be solved alone.**

---

## Lore Context

The mansion once belonged to a gifted family. When the dictator ordered the Purge, the clockmaker — a gifted survivor or trusted servant of the family — built a hidden tunnel connecting this apartment to the mansion grounds. It was an **escape route out**. Players use it **as an entrance in**.

The heroes' generation doesn't know gifted people existed. This puzzle is discovery — for both characters and players — delivered through objects, not cutscenes.

---

## Physical Layout

| Level | Player | What they find |
|---|---|---|
| Ground floor (workshop) | P1 | Large wall-mounted grandfather clock — open mechanism, no hands, no pendulum. Dozens of stopped clocks on shelves showing different times. |
| Attic / upper floor | P2 | Box containing clock hands + pendulum. Old newspapers, family portraits, clockmaker's journal fragments. Dumbwaiter access point. |

**Stairs are collapsed.** Players are separated the moment they enter. The **dumbwaiter** (wall-mounted kitchen lift) connects the two floors — it carries objects, not people.

---

## Beat 1 — Item Exchange

**Teaches:** *Passing objects between tiers via the dumbwaiter*

1. P1 examines the grandfather clock — missing hands and pendulum. The mechanism slots are visibly empty.
2. P2 finds the clock parts in an attic crate.
3. P2 loads the parts into the dumbwaiter and sends it down.
4. P1 receives the parts and installs them into the clock.

**Why it matters:** The dumbwaiter recurs as a verb throughout the level. This is the safe, low-stakes introduction.

---

## Beat 2 — Asymmetric Information

**Teaches:** *"I see something you don't — read it to me"*

1. The clock must be wound to a specific time. P1 sees only shelves of stopped clocks — all showing different times. No way to know which is correct.
2. P2 in the attic finds clues: newspaper clippings, the clockmaker's journal, family portrait inscriptions.
3. **The correct time** is hidden in P2's materials — e.g., a news article: *"Fires broke out across the quarter at 3:07 on the night of the Arrests."*
4. **Distractor:** Multiple newspaper clippings with different times cover different events. P2 must read context aloud; both players determine which article refers to the Purge.
5. P2 calls the time. P1 sets the clock hands.

**Lore delivered:** The Night of the Purge. The mansion family's fate. Why this tunnel was built. The clockmaker's identity as a gifted survivor.

**Design note:** The distractor should require genuine conversation — not just "it's 3:07", but "wait, this one says 11:40 for the curfew announcement, and this one says 3:07 for when the fires started — which do we need?" Let players reason together.

---

## Beat 3 — Simultaneous Activation

**Teaches:** *"On my count — go"*

1. P1 grips the clock's winding crank (hold input — sustained).
2. P2, from above, must drop the pendulum through a floor opening at the correct moment while P1 is winding.
3. A short timing window — miss it and both reset.
4. On success: the mechanism activates. A wall panel (bookshelf or section of wainscoting) slides open — the hidden room and tunnel entrance revealed.

**Design note:** The hold + drop timing should be forgiving on the first attempt, strict on retries — let players feel clever, not punished.

---

## Payoff Room

The hidden room contains:
- The clockmaker's journal — full lore, optional to read, worth it
- A single gas lantern still burning after 100 years (a small detail — someone built this to last)
- The tunnel entrance

**Atmosphere:** The tunnel should feel intentional, not desperate. Stone-lined, brass fixtures, maintained. This was built by someone who planned to come back. They never did.

---

## Systems Required

| System | Notes |
|---|---|
| `DumbwaiterComponent` | Replicated item container — two interactable endpoints (top/bottom). Reusable throughout the level. |
| Clock state machine | Tracks: hands installed? time set? winding active? Validates time against target value server-side. |
| Item actors (hands, pendulum) | Carryable props. Should be modeled clean — always visible in hand and during activation. Do not generate with AI tools. |
| `SimultaneityGate` | Two-player simultaneous hold trigger with configurable timing window. |
| Dumbwaiter UI | Minimal — show what's inside the lift from both sides |

---

## Open Questions

- [ ] Should the dumbwaiter have a visible rope/pulley P2 operates, or is it a simple "send" interaction?
- [ ] How long is the simultaneous window on Beat 3? (suggest: 1.5s first attempt, 0.8s after first fail)
- [ ] Does the journal auto-display or require P1/P2 to manually interact with it in the payoff room?
- [ ] Is the clockmaker's name established here, or left anonymous?

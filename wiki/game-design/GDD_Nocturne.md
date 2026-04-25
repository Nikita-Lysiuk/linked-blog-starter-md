---
type: gdd
title: Nocturne — Game Design Document
created: '2026-04-25'
updated: '2026-04-25'
tags:
  - gdd
  - game-design
  - nocturne
status: active
---

# Nocturne — Game Design Document

> **Studio:** Persival | **Genre:** Co-op Stealth + Puzzle | **Platform:** PC/Console | **Players:** 2 (local split-screen)

---

## Elevator Pitch

Two players share a split-screen nightmare. One steals faces. One rewrites memories. Neither can succeed alone — and every guard between them and the exit is a resource to exploit, not an obstacle to avoid.

---

## MDA Analysis

### Target Aesthetics (what players should *feel*)

| Half | Primary | Secondary |
|------|---------|-----------|
| **Stealth** | Fear, vulnerability, tension | Relief when successful |
| **Puzzle** | Curiosity, confusion → "aha" | Shared excitement |
| **Both** | Cooperation, trust in your partner | |

**Critical design constraint:** Players must never feel like gods. They should feel scared and careful — small animals moving through a space owned by bigger ones. No combat. Presence = eventual failure if not managed.

### Dynamics (what the mechanics produce)

- Pre-planning under uncertainty (P2 copies before P1 approaches)
- Asymmetric information (P2 sees the board, P1 is on the board)
- Cascading risk (one frozen guard creates a time clock for the whole team)
- Panic recovery (Paste A as a last resort when detection is seconds away)
- Social emergence (AIs react to each other, not just to players)

### Mechanics (what the rules are)

See: [[FaceSteal]], [[Telepathy]], [[AI_Social_Behavior]]

---

## Player Roles

### P1 — The Face-Stealer (operative)
- Works at **ground level**, close range
- **High risk, high reward** — must physically enter guard territory
- Tool: FaceSteal — steal an NPC's appearance and become them temporarily
- Constraint: tethered to frozen NPC, cannot stray far, time-limited

### P2 — The Telepath (observer/strategist)
- Works from **rooftops and elevated positions** via grapple hook
- **Low risk, high responsibility** — sees the whole board, guides P1
- Tool: Telepathy (Copy/Paste A/Paste B) — manipulate guard memories and states
- Constraint: must prepare before P1 acts (Copy first, Paste second)

### Why this split works
P2 has **information**, P1 has **action**. Neither is sufficient alone. P2 needs P1 to execute, P1 needs P2 to prepare and recover. The split-screen makes this literal — each player sees a different part of the world.

---

## Core Loop

```
P2 scouts patrol pattern from above
  → P2 copies target guard's consciousness (instant, undetected)
    → P1 approaches within FaceSteal range
      → P1 steals face (guard freezes, rope appears)
        → P1 moves through guard territory
          → P2 uses Paste B to redirect a blocking patrol guard
            → P1 reaches checkpoint / puzzle trigger
              → [Puzzle section]
                → Next stealth section
```

**Panic recovery branch:**
```
Guard spots P1 (AlertLevel rising)
  → P2 uses Paste A on that guard (restores pre-sighting memory)
    → AlertLevel resets, guard resumes patrol
      → P1 has ~2 seconds to hide during paste disorientation
```

---

## Win / Fail Conditions

| Condition | Result |
|-----------|--------|
| AlertLevel reaches 1.0 on any guard | Fail → restart from checkpoint |
| Social alarm: guard investigates frozen colleague, confirms something wrong | Fail → restart |
| P1 + P2 both reach exit | Level complete |

**Current implementation:** `APR_GameMode::TriggerFailure()` → `OpenLevel()` (checkpoint system deferred).

---

## Act Structure

### Act 1 — Tutorial (current focus)
- Simple environment: guards on clear routes, low density
- Each ability gets a dedicated "lightbulb moment":
  - Section 1: FaceSteal teaches the rope mechanic and time pressure
  - Section 2: Telepathy teaches Copy + Paste B (redirect a guard off P1's path)
  - Section 3: Both abilities together to solve a combined stealth + puzzle section
- No immunity mechanics yet (SuspicionLevel stays low throughout)
- Fail = restart (no checkpoint save needed yet)

### Act 2–3 (future)
- Immunity introduced: high SuspicionLevel makes guards resist abilities
- Officer detection: Officers check `Player.P1.Disguised` tag at high suspicion
- Timed puzzles alongside AI to combine both halves
- Checkpoint system active
- Option B on rope break (AlertLevel 0.5 instead of 0) appears on immune guards

---

## Level Structure (Act 1 template)

```
[Start] → Stealth A → [Checkpoint B] → Puzzle B → Stealth C → [Exit]
```

Each stealth section has a clear **spatial advantage** for P2: a balcony, rooftop edge, or walkway that lets them see below. P1 is always exposed on the ground.

---

## What's Missing Before Act 1 is Playable

- [ ] FaceSteal ability (P1) — [[PR-87]]
- [ ] Telepathy ability (P2) — [[PR-89]], [[PR-90]], [[PR-91]]
- [ ] Alert UI (visible indicator per guard)
- [ ] Rope UI (P1's FaceSteal tether)
- [ ] At least one puzzle mechanic
- [ ] A win condition trigger in the level

---

## Related
- [[FaceSteal]] — full mechanic spec
- [[Telepathy]] — full mechanic spec
- [[AI_Social_Behavior]] — alarm propagation, immunity
- [[AbilitySprint_Implementation]] — what to build next and in what order

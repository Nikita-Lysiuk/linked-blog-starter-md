---
type: mechanic
title: AI Social Behaviour — Alarm Propagation
created: '2026-04-25'
updated: '2026-04-25'
tags:
  - mechanic
  - ai
  - alarm
  - social
status: active
---

# AI Social Behaviour — Alarm Propagation

> Guards don't have a siren. They have a colleague who hasn't moved in too long.

---

## Design Intent

The alarm system is **social, not mechanical**. There's no button, no light, no sound cue. One guard notices something wrong with another guard and goes to check. That moment — visible to the player, actionable, terrifying — is the real alarm.

This keeps the aesthetic of vulnerability intact. Players aren't afraid of a system. They're afraid of a guard turning a corner at the wrong time.

---

## Frozen Colleague Detection

### How it works
`BTService_PR_CheckColleagues` runs on every AI every 0.5s:
1. Scans within `ColleagueCheckRadius` (default 1200 cm) for any AI in `Frozen` state
2. If found: starts accumulating `TimeObserved`
3. When `TimeObserved >= SuspiciousColleagueTime` (default 10 sec):
   - This AI's `AlertLevel` → 0.6 (above Investigate threshold of 0.5)
   - `LastKnownLocation` set to the frozen colleague's position
   - Normal BT Investigate branch activates: NoticeReaction → MoveToLastKnown → Search

### The staggered alarm (key design detail)
The guard doesn't instantly radio the whole building. They walk toward their frozen colleague first. **Other guards only get notified if the investigating guard gets close enough to confirm something is wrong.**

Propagation radius broadcast (when investigating guard reaches frozen colleague):
- All AIs within radius get +0.5 AlertLevel
- They all start searching their immediate area
- No central alarm — it radiates outward from the discovery point

This gives P1 and P2 a **window between "guard notices" and "everyone is searching"**.

---

## Alert States Summary

| AlertLevel | BT Branch | What player sees |
|-----------|-----------|----------------|
| 0–0.5 | Patrol / passive | Guard walking normally |
| 0.5–1.0 | Investigate | Guard suspicious, moving toward last known location |
| 1.0 | Failure | Level restart |

---

## Ability Immunity

When `UPR_SuspicionSubsystem::SuspicionLevel >= NPC.SuspicionThreshold`:

**FaceSteal immunity:**
- No rope UI shown (player's pre-read: this guard can't be stolen)
- If attempted anyway: rope appears, immediately snaps
- NPC wakes from `DisorientDuration` at **AlertLevel 0.5** (not 0)
- NPC remains immune permanently until SuspicionLevel drops

**Telepathy immunity:**
- Copy: always works (reading a mind leaves no trace)
- Paste: snapshot applies, but NPC immediately resists
- NPC wakes from `DisorientDuration` at **AlertLevel 0.5**

### Why this is good design
Immunity isn't a wall — it's a **cost**. Players who try abilities on immune guards don't hit an invisible barrier; they trigger a reaction. This teaches the mechanic through failure rather than a tooltip.

---

## SuspicionSubsystem (global counter)

`UPR_SuspicionSubsystem` persists across level sections (GameInstance subsystem).

| Event | Suspicion added |
|-------|----------------|
| Paste B expires (NPC snaps back) | +1 |
| Frozen colleague discovered + confirmed | +3 (future) |
| Immunity attempt | +0 (no penalty on attempt, only on wake) |

When SuspicionLevel crosses NPC's threshold → that NPC becomes immune. Different NPC types have different thresholds (Officers paranoid, Guards indifferent).

---

## Mechanic Interaction Map

```
P1 FaceSteal → NPC Frozen
                    ↓ (after SuspiciousColleagueTime)
             Nearby NPC investigates
                    ↓ (gets within 200 cm of frozen NPC)
             Broadcasts +0.5 AlertLevel radius
                    ↓
             All nearby guards search
                    ↓
             If not resolved → AlertLevel 1.0 → Failure
```

**Resolution paths:**
- P2 Paste A the investigating guard before they reach the frozen NPC (reset their curiosity)
- P1 returns to range (frozen NPC unfreezes before colleague arrives)
- P1 hides and lets search expire (AlertLevel decays to 0 after SearchDuration)

---

## Related
- [[FaceSteal]] — creates the frozen evidence
- [[Telepathy]] — Paste A can interrupt the investigation chain
- [[GDD_Nocturne]] — act structure and when immunity activates
- **Code:** `BTService_PR_CheckColleagues` — already implemented
- **Code:** `UPR_SuspicionSubsystem` — already implemented

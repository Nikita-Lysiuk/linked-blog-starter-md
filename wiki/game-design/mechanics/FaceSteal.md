---
type: mechanic
title: FaceSteal — P1 Ability
created: '2026-04-25'
updated: '2026-04-25'
tags:
  - mechanic
  - p1
  - facesteal
  - ability
status: active
owner: P1
---

# FaceSteal — P1 Ability

> P1 steals an NPC's appearance. The NPC freezes. P1 becomes them — same face, same faction, same identity — for as long as they stay close.

---

## Design Intent

FaceSteal is **borrowed time under a spatial constraint**. The player isn't powerful — they're a parasite tethered to the host. The longer they use it, the more evidence accumulates (frozen guard, rope stretching). The ability rewards planning and penalises greedy play.

---

## Full Behaviour Spec

### Activation
- P1 must be within `FaceStealRange` (config: default 1500 cm) of a valid target NPC
- Target must have `CanBeTargetedByAbility = true` in their `UPR_EnemyConfig`
- P1 presses ability input

### On Activation
1. P1's **SkeletalMesh** swaps to the target NPC's mesh (from their `UPR_EnemyConfig`)
2. Target NPC's `RuntimeTags` are **copied** onto P1 (e.g. `AI.Type.Guard`, `AI.Faction.Security`)
3. `Player.P1.Disguised` tag is applied to P1
4. P1's **team ID** changes from 0 → 1 (perception system now treats P1 as an ally)
5. Target NPC's `ConsciousnessState` → `Frozen`
6. `AI.State.Frozen` tag applied to target NPC
7. Rope UI appears between P1 and the frozen NPC (green)

### While Active (tick every 0.5s)
- Check distance from P1 to frozen NPC
- If distance > `FaceStealRange`:
  - Start **grace timer** (configurable, ~3–5 seconds)
  - Rope UI starts flashing red, faster as time runs out
- If distance returns within range before grace timer expires:
  - Cancel grace timer, rope returns to green

### On Grace Timer Expiry (rope snaps)
1. P1's mesh reverts to original
2. Stolen `RuntimeTags` removed from P1
3. `Player.P1.Disguised` tag removed from P1
4. P1's team ID reverts to 0
5. Frozen NPC enters `Overwritten` state (`DisorientDuration` ~1–2 sec)
6. During disorientation: NPC is unaware, **AlertLevel stays 0** — player's escape window
7. After disorientation: NPC resumes normal patrol at AlertLevel 0 (no memory of being frozen)

### On Manual Release (player presses ability again)
- Same flow as grace timer expiry

### Re-use
- P1 **can re-steal the same NPC** after the NPC has woken up and resumed patrol
- Levels are designed so this is required — abilities are the only way to progress
- No cooldown on re-steal (the NPC walking away and re-entering range is the natural gate)

---

## Immunity Interaction (Act 2–3)

When `SuspicionLevel >= NPC.SuspicionThreshold`:
- NPC is **immune** to FaceSteal
- No rope UI appears (player's read: no ability = not available)
- If P1 attempts activation anyway (edge case): rope appears and **immediately snaps**
  - NPC enters `Overwritten` disorientation (1–2 sec)
  - After disorientation: **AlertLevel → 0.5** (NPC starts searching)
  - NPC remains immune — no further ability attempts show UI

---

## Identity Layers

| What changes | When active | When dropped |
|---|---|---|
| P1 mesh | → NPC mesh | → restored |
| P1 team ID | → 1 (AI faction) | → 0 (player) |
| P1 tags | + stolen AI.Type.* | - removed |
| `Player.P1.Disguised` | applied | removed |
| NPC state | Frozen | Overwritten → Normal |

**Result while disguised:** Regular guards skip P1 on sight (same team). Officers check `Player.P1.Disguised` — immune at high SuspicionLevel.

---

## The Rope UI

The rope is the **emotional core** of this ability. It makes the invisible constraint visible.

| State | Appearance |
|-------|-----------|
| Within range, safe | Solid green line P1 ↔ NPC |
| Approaching edge | Green fading to yellow |
| Outside range, grace timer running | Red, flashing — faster as timer runs out |
| Immunity snap | Appears red, immediately breaks (< 0.5 sec) |

- Both players **see the rope** in split-screen
- P2 seeing the rope creates shared urgency — P2 knows P1's window without being told

---

## Balance Parameters (in `UPR_EnemyConfig`)

| Parameter | Default | Notes |
|-----------|---------|-------|
| `FaceStealRange` | 1500 cm | Radius to maintain disguise |
| `DisorientDuration` | 1.5 s | NPC wake-up window after rope snaps |
| `SuspiciousColleagueTime` | 10 s | How long frozen NPC goes unnoticed |

Grace timer duration (rope flash before snap) lives on `UFaceStealComponent` as a component property — not per-enemy-config since it's a player feel parameter.

---

## Open Design Questions (Act 2+)

- Can P1 steal an Officer? (Officer immune at high suspicion, but available at low — yes)
- Does wearing an Officer's face open doors a Guard face wouldn't? (Future)
- Can P1 steal a Civilian? (Probably not useful mechanically — Civilians don't patrol)

---

## Related
- [[Telepathy]] — P2's Copy must precede FaceSteal if Paste A recovery is needed
- [[AI_Social_Behavior]] — frozen NPC creates a social time clock
- [[GDD_Nocturne]] — full game context
- **Code:** `UFaceStealComponent` on `APR_BasePlayer` — PR-87

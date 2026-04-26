---
type: mechanic
title: FaceSteal — P1 Ability
created: '2026-04-25'
updated: '2026-04-27'
tags:
  - mechanic
  - p1
  - facesteal
  - ability
status: implementing
owner: P1
---

# FaceSteal — P1 Ability

> P1 steals an NPC's appearance. The NPC freezes. P1 becomes them — same face, same faction, same identity — for as long as they stay close.

---

## Design Intent

FaceSteal is **borrowed time under a spatial constraint**. The player isn't powerful — they're a parasite tethered to the host. The longer they use it, the more evidence accumulates (frozen guard, rope stretching). The ability rewards planning and penalises greedy play.

---

## Full Behaviour Spec

### Activation Input
- P1 presses ability input (E) → enters **Targeting Mode** via `IPR_PlayerAbility::StartTargeting()`
- P1 confirms target with LMB → `IPR_PlayerAbility::Activate()` fires

### Targeting Mode
- Screen edges darken, centre stays bright
- Glowing reticle VFX at screen centre (`BP_OnTargetingStarted()` → BP drives VFX)
- Line trace from camera centre begins (trace length = `UPR_FaceStealComponent::TraceDistance`)
- Valid NPC in crosshair → white outline appears (`BP_OnTargetFound(Target)`)
- P1 presses LMB on highlighted NPC to confirm

### On Activation (LMB confirm on valid NPC)
1. P1's **third-person SkeletalMesh** swaps to the target NPC's mesh (first-person hands unchanged)
2. Target NPC's `RuntimeTags` (e.g. `AI.Type.Guard`) **copied** onto P1's `RuntimeTags` (on `APR_BaseCharacter`)
3. Target NPC's `ConsciousnessState` → `Frozen` via `UPR_AIMemoryComponent::SetConsciousnessState()`
4. Rope UI appears between P1 and the frozen NPC (`BP_OnFaceStealActivated(Target)`)

### While Active
- Component checks distance from P1 to frozen NPC each tick
- If distance > `StealRadius`:
  - Start **grace timer** (`GraceTimerDuration`, default 3s)
  - `BP_OnRopeWarning(TimeRemaining, MaxTime)` fires each tick — BP drives rope pulse/colour
- If P1 returns within radius before grace timer expires:
  - Grace timer cancelled, `BP_OnEnterRadius()` fires — rope returns to normal

### On Grace Timer Expiry (rope snaps)
1. P1's mesh reverts to stored `OriginalMesh`
2. Stolen `RuntimeTags` removed from P1
3. `IPR_AIAbilityTarget::BeginDisorientation()` called on NPC — sets `bIsDisoriented` BB key
4. `BTTask_PR_Disorient` runs: NPC plays wake-up animation, waits `DisorientDuration` (~2s)
5. During disorientation: **AlertLevel stays 0** — player escape window
6. After disorientation: NPC resumes normal patrol, no memory of being frozen

### On Manual Release (player presses ability again)
- Same flow as grace timer expiry — `StopTargeting()` / `Deactivate()` called

### Re-use
- P1 **can re-steal the same NPC** after the NPC has woken up and resumed patrol
- Levels are designed so this is required — abilities are the only way to progress
- Cooldown on `UPR_FaceStealComponent` (default 15s) gates re-activation

---

## Immunity Interaction (Act 2–3)

When `UPR_SuspicionSubsystem::SuspicionLevel >= NPC.SuspicionThreshold`:
- NPC is **immune** to FaceSteal
- `CanActivate()` returns false — no targeting UI shown for immune NPC
- If P1 attempts anyway (edge case): rope appears, immediately snaps
  - `BeginDisorientation()` triggered on NPC
  - After disorientation: **AlertLevel → 0.5** (NPC starts searching)
  - NPC remains immune while Suspicion stays high

---

## Identity Layers

| What changes | While active | On deactivation |
|---|---|---|
| P1 third-person mesh | → NPC mesh | → OriginalMesh restored |
| P1 `RuntimeTags` | + stolen `AI.Type.*` tags | − removed |
| P1 team ID | unchanged (stays 0) | unchanged |
| NPC `ConsciousnessState` | `Frozen` | → `Normal` (after disorientation) |

**How disguise works:** AI perception still fires on P1 (TeamID 0). Disguise functions via tag check in `BTService_PR_UpdateAlertFromSight` — if perceived actor has `Ability.FaceSteal.Active` tag, AlertLevel delta is suppressed. Officers can do a secondary check on AI.Type.* for rank validation (Act 2+).

---

## The Rope UI

The rope is the **emotional core** of this ability — it makes the invisible constraint visible.

| State | Appearance |
|-------|-----------| 
| Within radius, safe | Solid green line P1 ↔ NPC |
| Approaching edge | Green fading to yellow |
| Outside radius, grace timer running | Red, flashing — faster as timer runs out |
| Immunity snap | Appears red, immediately breaks (< 0.5 sec) |

Both players **see the rope** in split-screen. P2 seeing the rope creates shared urgency — P2 knows P1's window without being told.

---

## Balance Parameters

| Parameter | Location | Default | Notes |
|-----------|----------|---------|-------|
| `TraceDistance` | `UPR_FaceStealComponent` | 2000 cm | Line trace reach in targeting mode |
| `StealRadius` | `UPR_FaceStealComponent` | 1500 cm | Range P1 must stay within while active |
| `GraceTimerDuration` | `UPR_FaceStealComponent` | 3 s | Rope flash window before snap |
| `CooldownDuration` | `UPR_FaceStealComponent` | 15 s | Time before ability can activate again |
| `DisorientDuration` | `UPR_EnemyConfig` | 1.5 s | NPC wake-up window after rope snaps |
| `SuspiciousColleagueTime` | `UPR_EnemyConfig` | 10 s | How long frozen NPC goes unnoticed by colleagues |

---

## Open Design Questions (Act 2+)

- Can P1 steal an Officer? (Immune at high suspicion, available at low — yes)
- Does an Officer's face open areas a Guard face wouldn't? (Future — tag-based area restrictions)
- Can P1 steal a Civilian? (Probably not useful — Civilians don't patrol)
- P2 Paste A on a Frozen NPC? (Breaks P1 disguise — needs cooperative design pass)

---

## Related
- [[Telepathy]] — P2 Copy must precede FaceSteal if Paste A recovery is needed
- [[AI_Social_Behavior]] — frozen NPC creates a social time clock for colleagues
- [[Consciousness State Machine]] — Frozen state technical detail
- [[GDD_Nocturne]] — full game context
- **Code:** `UPR_FaceStealComponent` (`Public/Mechanics/Abilities/PR_FaceStealComponent.h`) — PR-87

---
type: session
title: "FaceSteal — VFX, UI & Niagara Session"
date: "2026-06-15"
tags: [session, facesteal, vfx, ui, niagara, ue5]
jira: [PR-97, PR-98]
---

# FaceSteal — VFX, UI & Niagara Session

**Date:** 2026-06-15
**Scope:** Post-process outline, HUD widget + materials, C++ Niagara integration, Niagara rope base setup

---

## Completed

### 1. NPC Outline — `M_FaceSteal_Outline`

Post-process material that draws a silhouette outline on the targeted NPC.

- Material Domain: Post Process · Blendable Location: **Before DOF** (Before Tonemapping removed in UE5.4+)
- Project Settings: Custom Depth-Stencil Pass = **Enabled with Stencil**

**Material graph:**
- Block A: `ScreenPosition(ViewportUV)` as base UVs
- Block B: `SceneTexture(CustomDepth)` → `InvSize` → 4 offset UVs → 4 `SceneTexture(CustomDepth)` samples
- Block C: 3×Max → Subtract(max_neighbors − current pixel) → Clamp(0,1) = silhouette mask
- Block D (occlusion): `SceneDepth(R) > CustomDepth(R)` via If node → Multiply with mask
- Block F: `Lerp(PostProcessInput0 RGB, OutlineColor, mask)` → Emissive Color

**BP hooks:**
- `BP_OnTargetFound`: Cast to `APR_BaseAI` → GetMesh → `SetRenderCustomDepthPass(true)`
- `BP_OnTargetLost` / `BP_OnFaceStealActivated`: `SetRenderCustomDepthPass(false)`
- PostProcessVolume: Infinite Extent, Blendables → `M_FaceSteal_Outline`

---

### 2. HUD Widget — `WBP_FaceStealHUD`

Full cooldown + state-feedback widget. Two materials drive the visual.

#### Icon
- Generated on ideogram.ai: face split in half (left = smoke, right = silhouette)
- PNG, transparent background, Texture Group = UI

#### `M_AbilityIcon_Linear` — linear fill bottom-to-top
- Domain: User Interface · Blend Mode: Translucent
- `TexCoord → G → OneMinus → If(< FillAmount → white, >= → grey)` × TextureSample Alpha → Emissive
- TextureSample Alpha → Opacity

#### `M_AbilityIcon_Circular` — arcane ring with circular fill

| Parameter | Type | Default |
|---|---|---|
| FillAmount | Scalar | 0.0 |
| RingRadius | Scalar | 0.4 |
| RingSharpness | Scalar | 50 |
| NoiseScale | Scalar | 8 |
| NoiseSpeed | Scalar | 0.5 |
| NoiseStrength | Scalar | 0.02 |
| ColorDark | Vector | (0.02, 0.01, 0.05) |
| ColorLight | Vector | (0.3, 0.1, 0.8) |
| GlassSharpness | Scalar | 1.1 |
| GlassIntensity | Scalar | 0.4 |
| RotationAngle | Scalar | 0.0 |
| BreathAmount | Scalar | 0.0 |
| BreathSpeed | Scalar | 3.0 |

**Material graph highlights:**
- Angle: `TexCoord-0.5 → Arctangent2 → /2PI → +0.5 → Frac`
- Ring mask: `Length(UV-0.5) + Noise×NoiseStrength → Subtract(RingRadius) → Abs → OneMinus → Power(50) → Clamp`
- Breathing: `Time × BreathSpeed → Sine × BreathAmount → Add to RingRadius`
- Obsidian color: UE5 Noise normalized (it returns -1..1 not 0..1) → `Lerp(ColorDark, ColorLight)` + glass highlights via `Power + Multiply GlassIntensity`

#### Widget hierarchy
```
Canvas Panel → Overlay (200×200, bottom-center)
  ├── Background (Rounded Box, dark purple ~0.05,0.02,0.08)
  ├── RingMaterial (200×200) → M_AbilityIcon_Circular
  └── CooldownFill (170×170) → M_AbilityIcon_Linear
```

#### Event Graph
- **Event Construct:** Create Dynamic Material Instance for Ring + Fill → `SetBrushFromMaterial`
- **OnCooldownUpdated(TimeRemaining, MaxTime):** `FillProgress = 1-(TimeRemaining/MaxTime)` → SetScalarParameter `FillAmount` on both materials
- **OnCooldownExpired:** FillAmount=1, play `Anim_CooldownReady` (scale pulse + opacity blink)
- **OnActivated:** `BreathAmount=0.03`, brighter active colors
- **OnDeactivated:** `BreathAmount=0`, default colors
- **OnRopeWarning:** ColorLight/ColorDark → red
- **OnEnterRadius:** restore normal colors

#### Splitscreen wiring
- Widget created with **0.1s Delay** — workaround: P2 controller not yet possessed at construct time
- **`AddToPlayerScreen`** (not AddToViewport) + correct Owning Player via `PR_PlayerController` Cast

---

### 3. C++ Niagara Integration — `UPR_FaceStealComponent`

New header fields:
```cpp
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Face Steal|Effects")
TObjectPtr<UNiagaraSystem> RopeEffectTemplate;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Face Steal|Effects")
TObjectPtr<UNiagaraComponent> RopeEffect;

void createRopeEffect();
void snapRopeEffect();     // called from OnGraceTimerExpired
void retractRopeEffect();  // called from Deactivate_Implementation
```

**createRopeEffect:** `UNiagaraFunctionLibrary::SpawnSystemAttached` on owner root, then `SetVariableVec3("User.RopeStart")` + `SetVariableVec3("User.RopeEnd")`

**snapRopeEffect / retractRopeEffect:** Both call `RopeEffect->DeactivateImmediate()` + null the pointer. Different animations will diverge later.

**TickComponent update:**
```cpp
if (RopeEffect && (State == Active || State == GraceTimer))
{
    RopeEffect->SetVariableVec3("User.RopeStart", Owner->GetActorLocation());
    RopeEffect->SetVariableVec3("User.RopeEnd", StealTarget->GetActorLocation());
}
```

**Call order in state transitions:**
- `Activate_Implementation` → `createRopeEffect()` before `BP_OnFaceStealActivated`
- `Deactivate_Implementation` → `retractRopeEffect()` before `BP_OnFaceStealDeactivated`
- `OnGraceTimerExpired` → `snapRopeEffect()` before `Deactivate_Implementation`

**2D distance check — height ignored by design:**
`FVector::DistSquaredXY(...)` used in `UpdateStealRadius()` and `UpdateRopeWarning()`.

---

### 4. Niagara Rope — `NS_FaceStealRope` (base working)

Built from Static Beam template, emitter `E_CoreBeam` with Ribbon Renderer.

| Setting | Value |
|---|---|
| Curve Tension | 0.8 |
| Beam Emitter Setup | Absolute Start/End |
| Spawn Count | 25 |
| Lifetime | 5.0 |
| Beam Width | 8.0 (constant) |
| Color | Purple (0.3, 0.05, 0.8) |
| Jitter Position (Particle Update) | Amount=3.0, Delay=-0.05 |

**User Parameters exposed to C++:**
| Parameter | Type | Default |
|---|---|---|
| User.RopeStart | Position/Vector | (0,0,0) |
| User.RopeEnd | Position/Vector | (500,0,0) |
| User.RopeColor | Linear Color | (0.3, 0.05, 0.8, 1.0) |
| User.RopeWidth | Float | 8.0 |
| User.NoiseStrength | Float | 3.0 |

Rope appears between player and NPC on activation, tracks positions each tick, disappears on deactivation. ✅

---

## In Progress

### Niagara Rope — next steps

**Ribbon material** (currently DefaultRibbonMaterial / pink):
- Need custom material: purple glow, noise texture, soft edge bloom

**Additional emitters planned:**
1. **Fiber Wisps** (~10-15 sprites) — wisps drifting around the beam, fire-like drift + noise
2. **Energy Flow** (~5-8 points) — travel NPC → player along beam
3. **Anchor VFX** — magical claw/hook at attachment points, not a circle

**Rope states to implement:**
| State | Behavior |
|---|---|
| Deploy | `DeployProgress` 0→1 over ~0.3s, rope "shoots" player → NPC like a whip |
| Active | Stable purple, wisps active, energy flow running |
| GraceTimer | Color → red, noise increases, wisps faster |
| Snap | Splits at midpoint, each half retracts to its anchor, dissolves over 0.5s |
| Cancel | `DeployProgress` 1→0, rope retracts NPC → player |

**Performance budget:** ~40-50 total particles, GPU sim for wisps, simple material.

**Sag:** `SagAmount` parameter (0=straight, 1=heavy sag) — not yet implemented.

---

## Remaining VFX

| Feature | State |
|---|---|
| Targeting vignette (dark smoke on screen edges) | To Do |
| Activation smoke burst flying toward NPC | To Do |
| NPC freeze flash at steal moment | To Do |
| Frozen NPC visual indicator (frost shader or particles) | To Do |

---

## Sounds Needed

| Event | Sound type |
|---|---|
| Targeting enter | Subtle whoosh / dark magic activation |
| Target found | Soft lock-on confirmation |
| Activation (steal) | Aggressive magic burst |
| Active ambient | Dark hum / ethereal drone |
| Grace timer warning | Tension build, heartbeat pulse |
| Rope snap | Violent break / crack |
| Manual cancel | Reverse whoosh / retract |
| Cooldown finished | Ready chime / power restored |

Free SFX sources: freesound.org (CC0), zapsplat.com, Fab.com magic packs, pixabay.com/music.

---

## Key Technical Learnings

| # | Learning |
|---|---|
| 1 | UE5.4+ removed Before Tonemapping — use Before DOF |
| 2 | Custom Depth for post-process outline + SceneDepth for occlusion |
| 3 | UE5 Noise node returns -1..1, not 0..1 — normalize before Lerp |
| 4 | `Power(x, n)` with large n isolates values near 1 — use for ring sharpness |
| 5 | Splitscreen widgets: `AddToPlayerScreen` + correct Owning Player + 0.1s Delay for P2 |
| 6 | `DistSquaredXY` for 2D distance check ignoring Z |
| 7 | Ribbon Renderer is always billboard (faces camera) — reads as rope, not flat tape |
| 8 | Niagara stack order: Emitter Spawn/Update → Particle Spawn/Update → Render |
| 9 | Jitter Position must be in Particle Update (not Spawn) for continuous tremor |
| 10 | Update Beam must be above Jitter Position in the module stack |
| 11 | `DeactivateImmediate` kills particles instantly; `Deactivate` lets them live out their lifetime |
| 12 | `SetVariableVec3` for Vector params; `SetVariablePosition` for Position type (LWC precision) |

---

## Related

- [[game-design/mechanics/FaceSteal]] — mechanic spec
- [[game-design/technical-architecture/FaceSteal_VFX]] — VFX architecture reference
- [[entities/UPR_FaceStealComponent]] — component entity page
- [[sessions/2026-05-16_FaceStealComponent_CPP]] — C++ implementation session

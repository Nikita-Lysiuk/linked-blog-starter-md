---
type: technical-architecture
title: "FaceSteal — VFX & UI Architecture"
updated: "2026-06-15"
tags: [vfx, ui, niagara, materials, facesteal, ue5]
status: in-progress
---

# FaceSteal — VFX & UI Architecture

Full reference for all visual and UI systems that wrap the `UPR_FaceStealComponent` C++ logic.

---

## Asset Inventory

| Asset | Type | Status |
|---|---|---|
| `M_FaceSteal_Outline` | Post-Process Material | Done |
| `M_AbilityIcon_Linear` | UI Material | Done |
| `M_AbilityIcon_Circular` | UI Material | Done |
| `WBP_FaceStealHUD` | UMG Widget | Done |
| `NS_FaceStealRope` | Niagara System | In Progress |
| `M_FaceStealRope` | Material (for Ribbon) | To Do |
| Targeting vignette | Post-Process or Widget | To Do |
| Activation smoke burst | Niagara | To Do |
| Frozen NPC indicator | Shader / Niagara | To Do |

---

## NPC Outline — `M_FaceSteal_Outline`

Draws a silhouette on the NPC being targeted. Visible through geometry (occlusion-aware).

**Setup:**
- Material Domain: Post Process
- Blendable Location: Before DOF (Before Tonemapping was removed in UE5.4+)
- Project Settings → Custom Depth-Stencil Pass: **Enabled with Stencil**
- PostProcessVolume: Infinite Extent, add `M_FaceSteal_Outline` to Blendables

**Material graph:**
```
A: ScreenPosition(ViewportUV) → base UVs
B: SceneTexture(CustomDepth) + InvSize offsets → 4 neighbor depth samples
C: 3×Max(neighbors) - current pixel → Clamp(0,1) = silhouette mask
D: SceneDepth > CustomDepth via If → occlusion multiply (occluded = dimmer outline)
F: Lerp(PostProcessInput0, OutlineColor, mask) → Emissive Color
```

**Lifecycle (Blueprint):**
- `BP_OnTargetFound` → GetMesh → `SetRenderCustomDepthPass(true)`
- `BP_OnTargetLost` or `BP_OnFaceStealActivated` → `SetRenderCustomDepthPass(false)`

---

## HUD Widget — `WBP_FaceStealHUD`

Single widget per player. Shows ability state: cooldown progress, active breathing, warning color, ready pulse.

### Widget Hierarchy
```
Canvas Panel
└── Overlay (200×200, anchor: bottom-center)
    ├── Background — Rounded Box, color (0.05, 0.02, 0.08)
    ├── RingMaterial (200×200) — M_AbilityIcon_Circular instance
    └── CooldownFill (170×170) — M_AbilityIcon_Linear instance
```

### Splitscreen Wiring
- Create widget with **0.1s Delay** — P2 controller not possessed yet at BeginPlay
- **`AddToPlayerScreen`** (not AddToViewport) with Owning Player = `PR_PlayerController`

### State → Material Mapping

| Event | Parameter Change |
|---|---|
| OnCooldownUpdated(T, Max) | FillAmount = 1-(T/Max) on both materials |
| OnCooldownExpired | FillAmount=1, play `Anim_CooldownReady` |
| OnActivated | BreathAmount=0.03, brighter ColorLight |
| OnDeactivated | BreathAmount=0, default colors |
| OnRopeWarning | ColorLight/ColorDark → red tones |
| OnEnterRadius | Restore normal colors |

---

## Materials

### `M_AbilityIcon_Linear`

Bottom-to-top fill for cooldown progress.

- Domain: User Interface · Blend Mode: Translucent
- `TexCoord.G → OneMinus → If(< FillAmount → white, else → grey)` × TextureSample Alpha → Emissive
- TextureSample Alpha → Opacity

### `M_AbilityIcon_Circular`

Arcane ring with circular sweep fill and obsidian color animation.

**Angle calculation:** `TexCoord - 0.5 → Arctangent2 → /2PI → +0.5 → Frac`

**Ring mask:** `Length(UV-0.5) + Noise×NoiseStrength → Subtract(RingRadius) → Abs → OneMinus → Power(RingSharpness) → Clamp`

**Circular fill:** `If(angle < FillAmount → 0 else → 1)` × ring mask

**Obsidian color:** UE5 Noise (−1..1) normalized → `Lerp(ColorDark, ColorLight)` + `Power(x, GlassSharpness) × GlassIntensity`

**Breathing (Active state):** `Sin(Time × BreathSpeed) × BreathAmount` added to RingRadius

| Parameter | Default |
|---|---|
| FillAmount | 0.0 |
| RingRadius | 0.4 |
| RingSharpness | 50 |
| NoiseScale | 8 |
| NoiseSpeed | 0.5 |
| NoiseStrength | 0.02 |
| ColorDark | (0.02, 0.01, 0.05) |
| ColorLight | (0.3, 0.1, 0.8) |
| GlassSharpness | 1.1 |
| GlassIntensity | 0.4 |
| RotationAngle | 0.0 |
| BreathAmount | 0.0 |
| BreathSpeed | 3.0 |

---

## Niagara Rope — `NS_FaceStealRope`

Beam connecting player to targeted NPC. C++ drives start/end positions every tick.

### Current Setup (working)

- Built from Static Beam template
- Emitter `E_CoreBeam`, Ribbon Renderer (always billboard — reads as rope not tape)
- Curve Tension: 0.8 · Spawn Count: 25 · Lifetime: 5.0
- Beam Width: 8.0 · Color: purple (0.3, 0.05, 0.8)
- Jitter Position in **Particle Update** (not Spawn): Amount=3.0, Delay=-0.05
- Update Beam module must sit **above** Jitter Position in the module stack

### User Parameters (bound to C++ via `SetVariableVec3`)

| Parameter | Type |
|---|---|
| User.RopeStart | Position/Vector |
| User.RopeEnd | Position/Vector |
| User.RopeColor | Linear Color |
| User.RopeWidth | Float |
| User.NoiseStrength | Float |

> Use `SetVariableVec3` for Vector params. Use `SetVariablePosition` for Position type (LWC precision).

### Planned Emitters

| Emitter | Count | Purpose |
|---|---|---|
| Fiber Wisps | 10-15 sprites | Drift/noise around beam, fire-like motion — GPU sim |
| Energy Flow | 5-8 points | Travel NPC → player along beam axis |
| Anchor VFX | burst | Magical claw/hook at attachment points |

### Rope State Machine (to implement)

| State | Visual |
|---|---|
| Deploy | `DeployProgress` 0→1 over 0.3s — whip-like shoot to NPC |
| Active | Stable purple, wisps on, energy flow on |
| GraceTimer | Red tint, noise up, wisps faster |
| Snap | Split at midpoint, both halves retract to anchors, dissolve 0.5s |
| Cancel | `DeployProgress` 1→0, rope retracts back to player |

**SagAmount** parameter planned (0=straight, 1=heavy sag) — not yet built.

### Performance Budget
~40-50 total particles · GPU sim for wisps · Simple ribbon material (no heavy math)

---

## Remaining Work

### Ribbon Material — `M_FaceStealRope`
Currently using DefaultRibbonMaterial (pink). Needs:
- Purple emissive glow
- Noise/distortion texture
- Soft edge bloom

### Targeting Vignette
- Dark smoke on screen edges when Targeting state is active
- Approach: post-process material toggle or widget overlay

### Activation Flash
- Smoke burst gathering and flying toward NPC on activation
- Flash on NPC mesh at freeze moment

### Frozen NPC Indicator
- Frost shader overlay or particle cloud on NPC when ConsciousnessState = Frozen

---

## Sounds — `PR-99`

| Event | Sound Type |
|---|---|
| Targeting enter | Subtle whoosh / dark magic activation |
| Target found | Soft lock-on confirmation |
| Activation | Aggressive magic burst |
| Active ambient | Dark hum / ethereal drone |
| Grace timer warning | Tension build, heartbeat pulse |
| Rope snap | Violent crack / break |
| Manual cancel | Reverse whoosh / retract |
| Cooldown ready | Ready chime / power restored |

Free SFX: freesound.org (CC0), zapsplat.com, Fab.com magic packs.

---

## Related

- [[sessions/2026-06-15_FaceSteal_VFX_UI_Niagara]] — session that built all of this
- [[entities/UPR_FaceStealComponent]] — component fields and C++ interface
- [[game-design/mechanics/FaceSteal]] — mechanic spec

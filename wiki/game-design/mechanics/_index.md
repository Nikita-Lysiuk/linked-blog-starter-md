---
type: domain
title: "Mechanics Index"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - mechanics
  - game-design
status: seed
page_count: 0
---

# Mechanics

> One note per game mechanic. Each note links to a Technical Implementation in `technical-architecture/`.

## All Mechanics

_None yet. Template: `_templates/mechanic.md`_

---

## Mechanic Status Values

| Status | Meaning |
|--------|---------|
| `concept` | Idea stage, not designed |
| `designed` | Spec written, not implemented |
| `prototyped` | Working in isolation |
| `integrated` | In the build, not polished |
| `shipped` | Final, in released build |
| `cut` | Removed from scope |

## Cross-Link Rule

> Every mechanic note **must** contain a `Technical Implementation` section linking to `wiki/game-design/technical-architecture/[MechanicName]-impl.md`.
> If no impl note exists yet, create a stub. A mechanic without an impl link is tagged #orphan.

## Related
- [[wiki/game-design/technical-architecture/_index]] — implementations
- [[wiki/ecs/_index]] — ECS systems for mechanics
- [[wiki/patterns/_index]] — patterns used

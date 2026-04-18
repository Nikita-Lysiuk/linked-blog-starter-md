---
type: meta
title: "Central Command Dashboard"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - dashboard
status: developing
---

# Central Command Dashboard

> Live view of project state. Queries require **Dataview** plugin (enabled) or **Bases** (Obsidian v1.9.10+).
> Reload this page after making changes to see updated results.

---

## Active Sprint Tasks

### Project Persival — Active

```dataview
TABLE status, priority, updated
FROM "wiki/game-design"
WHERE contains(tags, "sprint") OR contains(tags, "persival")
SORT priority ASC, updated DESC
```

### Learning Campus — Active

```dataview
TABLE status, complexity, updated
FROM "wiki/concepts"
WHERE contains(tags, "learning-campus") OR status = "developing"
SORT updated DESC
```

---

## Feature Readiness — Mechanics

```dataview
TABLE status, loop_tier, updated
FROM "wiki/game-design/mechanics"
SORT status ASC, updated DESC
```

| Status | Meaning |
|--------|---------|
| `concept` | Idea stage |
| `designed` | Spec written |
| `prototyped` | Working in isolation |
| `integrated` | In the build |
| `shipped` | Final |

---

## Technical Debt

### #fixme — Critical

```dataview
TABLE file.link, tags, updated
FROM "wiki"
WHERE contains(tags, "fixme")
SORT updated DESC
```

### #refactor — Scheduled

```dataview
TABLE file.link, tags, updated
FROM "wiki"
WHERE contains(tags, "refactor")
SORT updated DESC
```

---

## Orphan Pages

> Notes tagged `#orphan` — not yet connected to the GDD or Learning Campus.

```dataview
TABLE file.link, type, status
FROM "wiki"
WHERE contains(tags, "orphan")
SORT type ASC
```

---

## Quick Access Grid

| Domain | Hub | Key File |
|--------|-----|----------|
| Game Design | [[game-design/_index]] | [[Disguise Steal]] |
| Ability Systems | [[game-design/technical-architecture/_index]] | [[Ability System Interface]] |
| Universal Ability | [[Universal Ability System Interface]] | [[Disguise Steal]] |
| Pattern Library | [[patterns/_index]] | [[ECS]] |
| ECS Registry | [[ecs/_index]] | [[ecs/entities/_index]] |
| Reference Library | [[sources/Reference Library]] | [[sources/_index]] |
| Learning: Collisions | [[concepts/Collisions]] | [[concepts/_index]] |
| Architecture Decisions | [[decisions/_index]] | [[meta/logbook]] |
| Technical Debt | [[meta/technical-debt]] | — |

---

## Project Kanban Boards

→ Project Persival board: [[kanban-persival]]
→ Learning Campus board: [[kanban-campus]]

---

## Recent Activity

```dataview
TABLE file.link, type, status, updated
FROM "wiki"
WHERE updated = date(today) OR updated >= date(today) - dur(7 days)
SORT updated DESC
LIMIT 15
```

---

## All Notes by Status

```dataview
TABLE file.link, type, updated
FROM "wiki"
WHERE type != "meta"
SORT type ASC, status ASC
```

---

## Notes

> **Bases alternative:** For Obsidian v1.9.10+, use `wiki/meta/dashboard.base` for native database views — no plugin required.
> The `.base` file and this `.md` file are complementary: the `.base` gives board/gallery views; this file gives Dataview table queries.

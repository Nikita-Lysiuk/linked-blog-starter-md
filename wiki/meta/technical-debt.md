---
type: meta
title: "Technical Debt Tracker"
created: 2026-04-18
updated: 2026-04-18
tags:
  - meta
  - technical-debt
status: developing
---

# Technical Debt Tracker

> All notes tagged `#fixme` or `#refactor` surface here. Reviewed as part of sprint planning.

---

## Active #fixme Items

```dataview
TABLE file.link as "Note", tags, status, updated
FROM "wiki"
WHERE contains(tags, "fixme")
SORT updated DESC
```

*(Requires Dataview plugin. Replace with Bases query for Obsidian v1.9.10+)*

---

## Active #refactor Items

```dataview
TABLE file.link as "Note", tags, status, updated
FROM "wiki"
WHERE contains(tags, "refactor")
SORT updated DESC
```

---

## Resolved Items

| Note | Issue | Resolution | Date |
|------|-------|------------|------|
| _None yet_ | — | — | — |

---

## Debt Classification

| Tag | Severity | Description |
|-----|----------|-------------|
| `#fixme` | High | Known bug, broken logic, or critical gap |
| `#refactor` | Medium | Works but needs rework before scaling |
| `#stale` | Low | Information may be outdated |
| `#orphan` | Medium | Note not yet linked into the GDD/wiki |

## Review Protocol

1. At sprint start: review all `#fixme` items — fix or defer
2. At sprint end: review `#refactor` items — schedule or accept debt
3. Monthly: purge `#stale` items — either update or delete

## Related
- [[wiki/meta/logbook]] — where debt is often first noticed
- [[wiki/meta/dashboard]] — Central Command Dashboard
- [[wiki/questions/_index]] — open questions feeding debt items

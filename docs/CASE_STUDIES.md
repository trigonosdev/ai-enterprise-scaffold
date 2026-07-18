# Case Studies

This file is currently empty because this project hasn't had a drift
incident worth documenting yet. That will change — see `CLAUDE.md`'s
"Understanding AI Code Drift" section for why it's not a matter of if.

When it happens: write it down here, close to when it happened, while the
details are still fresh. A real case study is worth more to the next
session than any amount of generic advice in `CLAUDE.md` — it has this
project's actual file names, actual timeline, and actual evidence.

## Template for a new entry

Copy this structure for each incident:

```markdown
## [Short, specific title] (YYYY-MM-DD or a date range)

### What broke

[Concrete, specific. Name the actual files, the actual function names,
the actual behavior. "Auth was inconsistent" is not specific enough;
"three separate hardcoded role-name sets in core/auth.py, api/auth.py,
and RolesPanel.tsx, none aware the others existed" is.]

### How it was found

[What question got asked that no per-PR review had been asking? Often
this is the single most important part of the entry — it's the thing
worth repeating on a cadence, not the specific bug.]

### Why per-PR / per-session review didn't catch it

[Each individual change usually looked correct in isolation. Explain
why — this is what proves the incident is a structural gap, not a
one-off mistake by whoever wrote it.]

### What fixed it

[The actual fix, and — separately — the process/tooling change that
should prevent the *class* of problem from recurring, not just this
instance of it.]

### What this changed going forward

[Did this add a line to CLAUDE.md's checklist? A new CI gate? A new
row in the Security Review Trigger list? Point to the actual diff.]
```

## Worked example (illustrative only — not a real incident from this project)

<details>
<summary>Expand to see what a filled-in entry looks like</summary>

## Duplicate role-check logic across three files (2026-01-15)

### What broke

Three independent, hardcoded checks for "is this user an admin" existed
simultaneously: a Python `set` in `core/auth.py`, a second Python `set`
in `api/auth.py` covering an overlapping but not identical set of roles,
and an undocumented, duplicated copy of the same idea in a React
component (`RolesPanel.tsx`) that had silently drifted out of sync with
either backend version.

### How it was found

Not by any single PR review — each of the three was added in a separate
session, each looking locally correct. Found by a dedicated audit asking
"does the *whole* authorization model still make sense," not "does this
one PR's authorization check work."

### Why per-PR review didn't catch it

Each addition was, in isolation, a reasonable, well-scoped change: "add
a role check here." No single PR touched more than one of the three
locations, so no reviewer was ever looking at all three side by side.

### What fixed it

Replaced all three with one data-driven capability table, queried the
same way from every call site (backend and frontend, via a shared
generated type). Added a CI check that fails if a new hardcoded role-set
literal appears anywhere in the codebase.

### What this changed going forward

Added a "Security/Authorization Architecture Audit" cadence to
`CLAUDE.md`, tighter than the general quarterly audit, specifically
because this class of drift concentrates in auth-adjacent code and the
blast radius of getting it wrong is higher than an ordinary feature area.

</details>

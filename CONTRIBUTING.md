# Contributing

This document is the source of truth for **how this project is built** —
process and engineering conventions. [`CLAUDE.md`](CLAUDE.md) owns *why*
this discipline exists and the drift-prevention framework itself; this
file owns the concrete day-to-day rules. Read both before opening a PR.

---

## Before you build anything

1. Read [`CLAUDE.md`](CLAUDE.md) in full — not skimmed. Its
   Pre-Generation Checklist applies to every change, human-written or
   AI-assisted.
2. If this touches an existing architectural decision, check
   [`docs/architecture/decisions/`](docs/architecture/decisions/) first —
   don't re-litigate a decision that's already recorded, and don't
   silently contradict one either. If you're genuinely reversing a prior
   decision, write a new ADR marking the old one superseded (see that
   folder's README for the rule).
3. If you're not sure whether something needs an ADR: if a future
   session, reading only the code, would have no way to tell *why* it
   looks the way it does, it needs an ADR.

## Pull request checklist

Every PR should be able to answer these before merge:

```
[ ] Did I search for existing helpers before writing new ones? (CLAUDE.md
    Pre-Generation Checklist, item 1)
[ ] Does every new write path stamp the required audit columns?
[ ] Does every new module have a logger?
[ ] Are all new business thresholds named constants, not magic numbers?
[ ] Is every new function under this project's size guideline?
[ ] Does this PR touch anything on CLAUDE.md's Security Review Trigger
    list? If so, has that heavier review actually happened?
[ ] Does this need a new ADR, or does it contradict an existing one?
[ ] Is there a test? If not, is that explicitly called out and justified,
    not silently skipped?
```

## Commit messages

[Define this project's actual convention — e.g. Conventional Commits
(`feat:`, `fix:`, `docs:`), or a scope-prefixed style. Keep it here so
every session (AI or human) uses the same format without re-deriving it.]

## Testing standards

[Define once established:
- What's the test runner?
- Does the test suite hit a real database, or mocks? (If real — say so
  explicitly; a decision to test against a real database instead of
  mocks is exactly the kind of thing worth an ADR, since it's a real
  trade-off someone deliberately made.)
- What's the minimum coverage bar, if any, and where is it configured
  (so this document doesn't silently drift out of sync with the actual
  enforced number)?
]

## Documentation map

[As soon as more than one or two docs exist, add a table here mapping
"if you're changing X, also update Y" — the fastest way for this to go
stale is for it not to exist at all, so start it early even if it's
short.]

| If you touch... | Also update... |
|---|---|
| [example] A new environment variable | The env template file **and** the deployment doc, both, never just one |
| [example] Layer rules / naming conventions | `CLAUDE.md`'s Architecture Invariants section |
| A new architectural decision | A new ADR in `docs/architecture/decisions/` |

## Documentation authority (when two docs disagree)

Pick a rule and write it down before it's needed — e.g. "the schema/code
always wins over prose description," or "the most recently dated ADR
wins over an older one that doesn't reference it." Whatever the rule is,
state it explicitly here so a disagreement gets resolved by policy, not
by whoever argues more persuasively in the moment.

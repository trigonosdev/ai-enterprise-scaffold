# Code Audit Ledger

This is a living, dated record of every audit run against this codebase
under `CLAUDE.md`'s Audit Cadence — the quarterly cross-file audit, the
tighter security/authorization audit, and any ad-hoc audit prompted by a
real incident. It exists so "when did we last check this, and what did
we find" has a real, permanent answer instead of living only in closed
PRs and old chat transcripts.

**This is not a certification.** Entries here are self-audits — by this
project's own contributors and AI sessions, against this project's own
codebase. They are evidence you can show a customer, auditor, or
compliance reviewer that the work happens on a real cadence, not a
substitute for independent third-party audit or certification (SOC 2,
penetration testing, etc.) where those are actually required. Do not
overstate what an entry here means — an honest "here's what we checked
and what we found" is worth far more, to a sophisticated reader, than a
vague claim of being "audited."

---

## How to add an entry

Append — don't rewrite history. If a past finding turns out to have been
wrong, or a past fix gets reverted, say so in a *new* entry; don't edit
an old one to make it retroactively correct. The point of a ledger is
that it's honest about the past, including this project's own
mistakes in assessing itself.

```markdown
## [YYYY-MM-DD] — [Scope: e.g. "Quarterly full audit" / "Security &
authorization audit" / "Post-incident audit: <what happened>"]

**Baseline metrics** (skip anything not meaningful to this audit):
- Files: [count]
- Lines of code: [count, and by what tool/method]
- Test coverage: [%, if measured]

**Findings:**

| # | Severity | Description | Status |
|---|---|---|---|
| 1 | [High/Medium/Low] | [Specific — name the actual file/pattern] | [Open / Fixed in #PR / Deferred, and why] |

**What changed since the last audit of this scope:**
[A short narrative — what got better, what got worse, what's still
outstanding from last time.]
```

## Compliance framework mapping (optional — delete this section if not relevant)

If this project operates in a regulated industry, it can be worth
mapping findings to the relevant framework's actual control language —
not because a self-audit satisfies the framework's requirements by
itself, but because it makes the *evidence trail* legible to whoever
eventually runs the real audit. Two shapes this often takes:

- **SOC 2 Trust Services Criteria** — map a finding to the specific
  criterion it's evidence for/against (e.g. a fixed access-control gap
  maps to CC6.1).
- **Industry-specific examination criteria** (e.g. a regulator's
  market-conduct exam, a payments-industry standard) — map findings to
  the specific area they'd be checked in a real exam.

State plainly, once, at the top of this file (already done above) that
this mapping is a self-assessment aid, not a claim of certification —
and don't restate that caveat inconsistently across entries in a way
that could be read as walking it back later.

---

## Audit history

*(No entries yet. The first one gets added the first time `CLAUDE.md`'s
Audit Cadence actually runs — see that file for when.)*

# Documentation Map

Start here. This file exists so "which document is authoritative for X"
has one answer, not a guess — a genuinely small addition that pays for
itself the first time two docs disagree and nobody can tell which one
is stale.

| Document | What it contains |
|---|---|
| [`../CLAUDE.md`](../CLAUDE.md) | AI-assistant drift-prevention framework — read this first, always |
| [`../CONTRIBUTING.md`](../CONTRIBUTING.md) | Process and engineering conventions |
| [`../AI_PROMPTS.md`](../AI_PROMPTS.md) | Reusable prompt templates by task category |
| [`architecture/decisions/`](architecture/decisions/) | ADRs — why each significant decision was made |
| [`CASE_STUDIES.md`](CASE_STUDIES.md) | Real drift incidents from this project, once any exist |
| [`CI_CD_PATTERNS.md`](CI_CD_PATTERNS.md) | Test-gate and release-smoke-test patterns |

## Authority rule (fill this in early)

[State explicitly what wins when two documents disagree — e.g. "the
actual schema/code always wins over prose description," or "the most
recently dated ADR wins over an older one that doesn't reference it."
An unstated rule here means every disagreement gets resolved ad hoc,
which is its own quiet source of drift.]

## As this project grows

Add a row to the table above every time a new top-level doc is created.
A documentation map that doesn't list a real doc is worse than no map at
all — it actively misleads whoever trusted it.

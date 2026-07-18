# Architecture Decision Records

This folder contains the Architecture Decision Records (ADRs) for this
project — a permanent, append-only log of every significant technical
decision, written at the time each decision was made and never rewritten
after the fact.

---

## Why ADRs exist

An AI session generating code today has no memory of the reasoning behind
a decision made three sessions ago. Without a written record, an
unusual-looking pattern reads as a mistake to be "helpfully" fixed —
reverting a deliberate choice because the reasoning behind it was never
written down anywhere the next session could find it.

ADRs also protect against well-intentioned rework from human contributors
for the same underlying reason: when someone sees something that looks
like it could be "simplified," the ADR for that decision explains what
was considered and rejected, preventing the project from re-learning the
same lesson twice.

## Rules

1. **Numbered sequentially, never reused.** `0001`, `0002`, ... Even a
   rejected/abandoned decision keeps its number — do not renumber later
   ADRs to fill a gap.
2. **Never edit a merged ADR's Decision or Why sections after the fact.**
   If a decision is later reversed, write a *new* ADR and mark the old one
   "Superseded by ADR-NNNN" in its Status Notes section. The history of
   changing your mind is valuable information; overwriting it destroys
   that.
3. **One decision per ADR.** If a PR contains two genuinely separate
   architectural decisions, write two ADRs.
4. **Write it when the decision is made, not retroactively "for the
   record" months later.** Context degrades fast; the reasoning captured
   the day of the decision is far more trustworthy than a reconstruction
   after the fact.
5. **Use [`template.md`](template.md) for every new ADR.**

## Index

| ADR | Title | Status |
|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record Architecture Decisions | Accepted |

*(Add a row here every time a new ADR is merged. Keep this table
current — it's the fastest way for a new session to see what's already
been decided before proposing something that contradicts it.)*

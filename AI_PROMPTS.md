# AI Prompt Templates

`CLAUDE.md` owns the *why* — the drift-prevention policy, the checklist,
the audit cadence. This file owns the *copy-paste starting point* for
common task categories, so each new task begins from a prompt that
already encodes "search for existing patterns first," instead of a blank
page that optimizes purely for "make it work."

Add a new template here the second time you find yourself writing a
similar prompt from scratch — that's the signal it's a real category
worth templating, not a one-off.

Every template below is a starting skeleton. Fill in the `[bracketed]`
parts with this project's real specifics before using it.

---

## Template: New Backend Endpoint / Service

```
Implement [feature] in [layer/module].

Before writing any code:
1. Search for existing helpers that already do part of this — grep for
   [the concept], check [this project's known shared-helper locations].
2. Check CLAUDE.md's Architecture Invariants section for this project's
   layer rules and naming conventions before choosing where this code
   lives.

Requirements:
- [Functional requirements]
- Must stamp [this project's audit columns] on every write
- Must use [this project's logger convention]
- Must use [this project's pagination helper] if this is a list endpoint
- Error responses must follow [this project's error-mapping convention]

After writing the code, before calling it done:
- Run CLAUDE.md's Pre-Generation Checklist against your own diff
- Add a test — unit test if this is pure logic, integration test if it's
  a new security boundary or crosses a real database
```

## Template: Database Migration / Schema Change

```
Add [schema change] via a new migration.

Before writing any SQL:
1. Read this project's migration conventions (docs/architecture/decisions/
   or a dedicated DATABASE_STANDARDS.md if one exists).
2. Check whether this touches any table with row-level security or
   role-based grants already applied — if so, the new column/table needs
   the same treatment, not a silent gap.

Requirements:
- [Naming conventions for this project's migrations]
- If this adds a privileged/sensitive column, does it need an audit log
  entry, an encryption requirement, or a masked-by-default read path?
- Is this migration idempotent / safe to re-run?

After writing it:
- Actually run it against a genuinely empty database, not just an
  already-migrated one — a migration that only works incrementally on an
  existing dev database has not been proven to work on a fresh install.
```

## Template: Security Review

```
Review [file(s)/PR] for security issues, specifically:

1. Does every new privileged action (auth check, data mutation touching
   another user's data, admin-only operation) have an explicit
   authorization check — not an implicit one that happens to work today?
2. Does every new guard clause (rate limit, environment check, permission
   check) have a test that proves it actually fires under the condition
   it claims to block? A guard nobody has verified fires is not a
   control.
3. Is there more than one place in this diff (or, cross-referenced
   against the existing codebase) answering the same "is this actor
   allowed to do X" question? If so, that's exactly the drift pattern
   CLAUDE.md warns about — flag it even if both current answers happen to
   agree today.
4. Does this diff touch anything on CLAUDE.md's Security Review Trigger
   list? If so, has the heavier review process for those files actually
   run, not just ordinary code review?

Report findings ranked by severity, each with: what's wrong, the
concrete scenario where it breaks, and the smallest fix that actually
closes the gap (not just silences the symptom).
```

## Template: Cross-Cutting Drift Audit

```
Audit [subsystem / whole codebase] for cross-file drift — this is
explicitly NOT a per-file correctness review, it's a "does this whole
area still make sense when looked at all at once" review. Per-PR review
structurally cannot see this; that's the entire reason this audit
exists.

Look specifically for:
1. The same concept (a helper, a validation rule, a permission check)
   implemented more than once, especially if the implementations have
   quietly diverged.
2. A design decision's own stated deferral condition ("premature until
   X") that has since been invalidated by a later change, without
   anyone connecting the two.
3. A capability/permission/scope with zero enforcing call sites anywhere
   in the codebase — defined but never actually checked.
4. Missing cross-cutting concerns at scale: "how many modules have zero
   logging," not "does this one module have logging."
5. A documented process (backup, deployment, migration rollback) that
   nobody has actually run end-to-end recently, if ever.

For every finding: name the actual files/lines, not a general
impression. A finding that can't be pointed at is not actionable.

Output as a real, filed list (issues/tickets), not just narrated in
chat — an audit whose findings live only in a conversation transcript
gets lost the same way institutional memory always does.
```

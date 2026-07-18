# [Project Name] — AI Assistant Guide

This file is loaded automatically by Claude Code (and similar tools) at the
start of every session. It is the primary control against AI code-generation
drift. Every developer — human or AI — working on this repository must read
and apply it before writing or reviewing any code.

**This file is a template**, distilled from lived experience running
real, AI-assisted engineering projects. Replace every `[bracketed]`
placeholder with content specific to *this* project. Do not leave
placeholders in place and call the file "done" — an unfilled template
provides none of the protection described below.

---

## Why This File Exists

AI-assisted development is a force multiplier, but it introduces a specific
failure mode that has no close analogue in purely human-written codebases:
**context-reset drift.**

Each AI session starts with no memory of prior sessions. The AI that
implements a new endpoint today has not "seen" the 40 endpoints written
before it. Without explicit controls, every session produces plausible,
locally-correct code that is subtly inconsistent with what came before:

- The same helper function reimplemented in several files with several
  divergent behaviors
- The same business threshold hardcoded as a magic number in multiple places
- Most modules with zero logging, because "add a logger" was never in the
  prompt
- A god file with no service layer, because each feature was added one PR at
  a time and nobody held the cumulative picture
- A privilege/authorization check hardcoded in one place, then quietly
  re-implemented three more times over the following month by three more
  sessions, none of which knew the other three existed

This file is meant to be the cumulative picture — encoding what "correct"
looks like for this specific project so each new session (AI or human) can
reattach to it instantly, instead of re-deriving it from scratch or, worse,
silently diverging from it.

---

## Quick Reference: Pre-Generation Checklist

Run this checklist **before writing any new code**. Do not skip steps to
move faster — skipping is how drift accumulates. Customize the specific
helper/pattern names below for this project; keep the *shape* of the
checklist even if you change every line inside it.

```
[ ] 1. Have I searched for this pattern already in the codebase?
        grep the key concept before writing a new helper.
        [List this project's known shared-helper locations here, e.g.:
         Lookup resolvers → app/api/_common.py
         Response mappers → look for existing _x_to_response functions
         Pagination → app/api/_pagination.py]

[ ] 2. Does this write path stamp required audit columns?
        [e.g. created_by / updated_by, or whatever this project's
        equivalent is — every model that has these columns must have
        them populated on every mutation.]

[ ] 3. Does this module have a logger?
        A structured logger at the top of every module that does I/O.
        A log line on every privileged mutation.
        A log line on every security-relevant event.

[ ] 4. Are exception types specific?
        Never a bare catch-all. Always the specific exception type(s)
        that can actually occur here.

[ ] 5. Are hardcoded values named constants?
        No magic numbers, no inline literal business thresholds,
        currencies, environment names, or credentials.
        Shared thresholds → a constant imported everywhere it's used,
        not copy-pasted.

[ ] 6. Is the function under [N] lines?
        If it's approaching that limit, it needs a service layer or
        sub-functions. [This project's convention: e.g. "Endpoint
        functions orchestrate; service functions do work."]

[ ] 7. Are there type annotations / type hints on every public signature?

[ ] 8. Does the error response match this project's own conventions?
        [Document the actual convention here once it exists — e.g.
        which failure maps to which HTTP status, or equivalent.]

[ ] 9. Does a partial-update path only touch fields the caller actually
        sent, not silently reset omitted fields to a default?

[ ] 10. Does this change have a test?
        A new service function needs a unit test.
        A new security boundary needs an integration test.
        If you cannot test it, say so explicitly rather than silently
        skipping it.
```

---

## Prompt Templates

If this project accumulates enough repeated task categories (backend
endpoint, frontend component, database migration, security review,
audit pass), put reusable prompt templates in a separate `AI_PROMPTS.md`
rather than growing this file indefinitely. Policy and the *why* live
here; copy-paste templates live there. See the starter
[`AI_PROMPTS.md`](AI_PROMPTS.md) in this scaffold for the pattern.

---

## Architecture Invariants

Fill this section in as soon as the project's real architecture exists —
an empty invariants section is worse than a short, honest, project-specific
one. These are meant to be non-negotiable structural rules: any session
that violates them is producing code that will be rejected at review,
regardless of whether the code "works."

### Layer Rules

```
[Define this project's actual layers and what may/may not cross between
them. Example shape, from a FastAPI + SQLAlchemy project:

app/api/      → Routing, request parsing, response serialization only.
                May raise HTTP-layer exceptions. Must not contain
                business logic.
app/services/ → Business logic. No HTTP-layer imports. Raises domain
                exceptions, translated to HTTP status at the API layer.
app/models/   → ORM only. No business logic. No service imports.
app/core/     → Infrastructure (auth, config, database, logging).
                Imported by all layers, imports nothing project-specific.
]
```

### Shared Helpers — Use These, Do Not Recreate Them

```
[This is the single highest-leverage section in the whole file, and the
one most likely to go stale. Every time a genuinely shared helper is
introduced, add it here, in this exact format, so the next session
finds it via grep before writing a duplicate:

_common.py:      get_x_or_404(db, id)
_pagination.py:  paginate(db, stmt, limit, offset) → (rows, total)
]
```

### Naming Conventions

```
[Document the actual conventions once established. Example shape:

Endpoint function:        verb_noun()           e.g. create_widget
Response mapper:          _x_to_response()       e.g. _widget_to_response
Fetch-or-404 helper:      _get_x_or_404()
Auth dependency:          actor: Actor            (not _actor, not user)
Private module constant:  _UPPER_SNAKE
Shared/public constant:   UPPER_SNAKE
]
```

### Mandatory Per-Module Pattern

```
[Whatever boilerplate every new module in this codebase must start
with — a logger, a specific import order, a docstring convention.
Keep this current; it's the thing new sessions copy-paste from.]
```

---

## Understanding AI Code Drift

### What It Is

AI code drift is the accumulated divergence between files when:
1. Each file is generated in an independent session with no memory of
   prior sessions
2. The generating prompt focuses on "make this feature work" rather than
   "is this consistent with what already exists"
3. No automated gate enforces cross-file consistency

The result is a codebase where the same concept has several
implementations, the same business rule has several slightly different
encodings, and a cross-cutting concern (logging, auditing, authorization)
exists in some fraction of the modules that need it. Each individual file
passes review. The codebase as a whole fails the smell test.

### Why It Is Not Obvious Until It Accumulates

When a single change is reviewed, a locally-duplicated helper looks
correct. A missing logger is a low-priority nit. A hardcoded value is
"just a number." None of these is a blocker individually.

After enough changes, the drift becomes visible only at the codebase
level — which is exactly the level individual PR review misses, and
exactly the level a dedicated, periodic, cross-cutting audit exists to
catch (see Audit Cadence below).

### The Three Root Causes

**1. Context window amnesia.** The AI session that implements feature B
has not read the code that implements feature A, even if A shipped last
week. It generates A's equivalent from scratch, producing a
locally-correct but globally-redundant result. No amount of "be careful"
in the prompt overcomes this — the prior context simply is not present.
The Pre-Generation Checklist above exists specifically to force a
search step before generation, substituting for memory the AI doesn't have.

**2. Optimization target mismatch.** A prompt that says "implement X"
optimizes for a working implementation of X. It does not optimize for
"first look for existing patterns and reuse them" unless that's made an
explicit, checked step. The checklist above exists to rewire the
optimization target, not to add friction for its own sake.

**3. No skin in the game.** An AI session that produces drift does not
maintain the code afterward. It does not accumulate the friction of
writing the same helper a fourth time. The feedback loop that would
naturally cause a human engineer to extract a shared helper ("I'm tired
of fixing this in four places") does not exist for a session with no
memory of the first three times. This is why a *human* architect,
periodically asking "does this whole subsystem still make sense" (see
Audit Cadence), remains necessary — that question has no natural trigger
inside any single generating session.

### Case Studies

The pattern behind real drift incidents, seen enough times to be worth
naming here even before this project has its own: they get found by
asking a different question than the one every prior session was asked.
Every generating session gets asked "does this feature work." Drift is
only visible to the question "does this whole subsystem, looked at all
at once, still make sense" — and separately, for deployment
infrastructure specifically, "has anyone actually run this, start to
finish, for real, from a genuinely empty starting state."

**As this project accumulates its own real drift incidents — and it
will; that is not a failure of this framework, it's why the framework
exists — record them.** Create `docs/CASE_STUDIES.md` the first time it
happens, and write down what broke, why per-PR review didn't catch it,
and what question finally surfaced it. A case study from *this* project,
with *this* project's actual file names and actual timeline, is more
useful to the next session than any generic template section —
including this one. The two patterns most worth watching for, based on
where this kind of thing tends to hide:

- **Authorization/permission logic** — the same "who is allowed to do X"
  question, answered more than once, in more than one place, none of
  them aware of the others.
- **The documented production deployment path** — a green test suite
  has never meant "the documented deployment actually works." Those are
  different claims. Nothing enforces the second one unless something
  explicitly, automatically does — see Deployment-Path Verification
  below.

### What Prevents It

In rough order of leverage:

1. **This file.** The pre-generation checklist forces adversarial
   pre-generation thinking. Prompt templates (`AI_PROMPTS.md`) encode
   the correct optimization target so it doesn't have to be re-specified
   every time.

2. **Automated gates.** Linting rules, type checking, and a CI check
   that fails if a known shared helper isn't being used where it should
   be. These catch drift without requiring anyone to remember to look.

3. **A periodic, dedicated, cross-cutting audit** — not per-PR review,
   which structurally cannot see codebase-level drift — explicitly
   scoped to pattern detection across the whole codebase. See Audit
   Cadence below.

4. **Human sign-off on shared infrastructure.** Any change to the
   project's actual shared-helper files, auth core, or equivalent
   requires deliberate human review. These are the seams that, if
   inconsistent, propagate inconsistency everywhere downstream.

5. **A tighter, risk-tiered audit for security/authorization and
   deployment-infrastructure subsystems specifically** — drift tends to
   concentrate in exactly these two areas, and both are categorically
   higher blast-radius than an ordinary feature area if they drift
   silently.

6. **An automated, end-to-end smoke test proving the actual deployment
   path works — not just that the application code passes tests.** A
   green test suite has never meant "the documented deployment actually
   works" unless something explicitly, automatically proves the second
   claim too.

---

## Verify, Don't Assume

This is a different failure mode from AI code drift, and it deserves its
own section rather than living as a footnote under Deployment-Path
Verification below. Drift is about cross-session memory loss producing
*inconsistent* code. This is about trusting a *reported* success —
a tool's exit code, a green check, a confident chain of reasoning —
instead of checking the actual resulting state. Both are AI-assistance
failure modes with no close human-only analogue at the same frequency;
both need an explicit countermeasure, not just good intentions.

### Where this bites hardest

1. **A tool's exit code or stdout is not the same claim as "the state is
   now what I intended."** A multi-file staging command that partially
   fails on one bad path can silently leave the rest unstaged, with
   nothing obviously wrong in the output — the fix isn't more careful
   reading of output, it's re-checking the actual state (`git status`,
   `git diff --cached`) before treating a step as done, every time, not
   just when something feels off.

2. **CI green is not the same claim as "the documented deployment
   works."** Covered in depth under Deployment-Path Verification below —
   the canonical instance of this principle, not the whole of it.

3. **An authenticated or local context is not the same claim as "this
   works for the person it's actually for."** Something can pass every
   automated check while being completely broken for the actual external
   user, if every check that ran had access the real user doesn't have.
   Before shipping anything meant to work for an outside party — a
   public download, an unauthenticated endpoint, a fresh signup flow —
   test it from that exact vantage point, not from inside whatever
   authenticated tooling built it. "It worked when I tried it" is only
   evidence if "I" was standing where the real user stands.

4. **Confident reasoning is not the same claim as empirical proof.**
   "This should be safe because X, Y, Z" is a hypothesis, not a result —
   even (especially) when it's airtight-sounding and produced under time
   pressure to move faster. Test it anyway when the cost of being wrong
   is real. The reasoning is still valuable — it's what tells you *what*
   to test and *why* a given check would be conclusive — it just isn't a
   substitute for running the test.

### The habit this requires

After every consequential action — a commit, a push, a merge, a deploy,
a claim that something "now works" — verify against the actual source of
truth, not the tool's own report of what it did:

- After staging changes, look at the actual staged diff before
  committing, not just the list of filenames.
- After pushing, confirm against the remote (a fresh `git log`/API call
  against the actual host), not just a lack of error output.
- After claiming a fix works, reproduce the original failure first if
  you haven't already — you cannot know a fix works if you never
  independently confirmed what "broken" looked like.
- Before declaring anything "done" that another party will rely on,
  articulate what vantage point actually matters (an external
  unauthenticated user? a fresh install? a cold cache?) and verify from
  *that* vantage point specifically — not the most convenient one.

None of this is about distrust of tools or of AI-generated output in
general — it's specifically about the gap between "a step reported
success" and "the thing that step was supposed to accomplish is actually
true," which is a real, recurring gap, not a hypothetical one.

### Who verifies matters, not just whether verification happened

**An AI session must not be both the sole author and the sole approver
of a change to a production or security-critical surface.** CI passing
is not human review, and an AI session's own confidence that a change is
safe is not human review either, no matter how thorough the session's
own testing was — the entire point of a second, independent reviewer is
catching what the first author's blind spots hid from them, and an AI
session reviewing its own work shares its own blind spots by definition.

This is a policy, not a preference:

- Any change touching a file on the Security Review Trigger list below,
  or a production deployment/database-configuration surface, requires an
  explicit human sign-off before merge — a real person looking at the
  actual diff, not a delegated "looks fine, ship it" given without
  reading it.
- If an AI session is about to merge, deploy, or otherwise finalize its
  own change to one of these surfaces with no human checkpoint in the
  loop, that is the moment to stop and ask, explicitly, rather than
  proceed on the strength of automated checks alone.
- "The user said go ahead earlier" does not retroactively cover a
  specific, consequential action later unless that action was actually
  named. Broad delegation ("handle it," "do everything") is not the same
  authorization as a specific yes to a specific, named, high-stakes step —
  ask again when the stakes step up, even if that feels like friction.

---

## Audit Cadence

### Quarterly (or milestone-boundary) Full Audit

**Who:** One engineer or AI session, dedicated time
**What:** Full cross-file pattern detection across the whole codebase —
not a single PR's diff
**Output:** Filed as issues/tickets against a real milestone, not just
narrated in chat — **and** a new dated entry in
[`docs/CODE_AUDIT.md`](docs/CODE_AUDIT.md), the permanent ledger. Issues
get closed and forgotten; the ledger is what proves, months later, that
this cadence actually ran.
**Trigger:** Start of each planning cycle, or after N feature PRs since
the last one, whichever comes first

Scope specific to this audit, not covered by ordinary PR review:
- Cross-file duplication of logic that should be one shared helper
- Missing cross-cutting concerns at scale ("how many modules have zero
  logging?" not "does this one module have logging?")
- Business-threshold drift (how many places does a given magic number
  appear as a literal?)
- Function/file size distribution across the codebase
- Dead code accumulation

### Security/Authorization Architecture Audit (tighter cadence)

The general audit above is not, by itself, scoped to answer "does the
authorization architecture, as a whole, still make sense" — that
question needs its own tighter cadence, because auth-adjacent files
accumulate hardcoded, drift-prone decisions faster than the rest of the
codebase, and the blast radius of getting one wrong is categorically
higher.

**Who:** A human architect leads or reviews it. An AI session may perform
the mechanical detection pass, but a human must synthesize the findings —
an AI audit can mis-categorize or hallucinate a finding, and only a human
owner can judge whether a flagged pattern is real drift or acceptable
variation.

**Cross-model requirement, if practical:** if most of the code under audit
was generated by one model family, run this audit with a different model
family where possible. A model auditing its own kind of output shares that
output's blind spots.

**What to look for:**
- More than one hardcoded identity/role/permission check answering a
  similar question
- A design decision's own stated deferral condition that a later change
  has since invalidated, without anyone connecting the two
- A permission/capability/scope with zero enforcing call sites anywhere
- A backend authorization surface with no corresponding admin UI, or a
  frontend check with no backend enforcement behind it — either direction
- A guard clause that exists to block a dangerous action, with no test
  proving it actually blocks it — a guard nobody ever verified fires is
  not a control, it's documentation shaped like one

**Output:** same as the general audit — real filed issues, plus a dated
entry in `docs/CODE_AUDIT.md` recording what this specific pass covered.

### Deployment-Path Verification

**The canonical instance of "Verify, Don't Assume" above, specific to
deployment.** A fully green CI pipeline has never meant "the documented
production deployment actually works" — those are different claims.
Application tests validate application code against a test database set
up in whatever way is fastest for CI, which is very often *not* the same
way the documented production deployment sets one up. Nothing enforces
the second claim unless something explicitly does.

**Minimum bar:** an automated job, run on every release (not just
occasionally by hand), that stands up the actual documented deployment
mechanism from a genuinely empty starting state and asserts a real health
check — not "containers started," an actual "the application inside is
correctly serving requests" check — before a release is considered done.

**Extend this to the actual consumer's vantage point, not just your
own.** If the deployment mechanism is meant to work for an external
party (a downloaded installer, a public package, a pulled container
image), the smoke test above needs to run from an equivalently
unauthenticated, external position — not from inside CI's own
authenticated tooling, which can have access a real outside user simply
doesn't. A green smoke test that only ever ran authenticated proves
less than it looks like it proves.

---

## Per-Feature Review (Before Merge)

Every PR that touches core application logic should answer these
questions in the PR description or a review comment — adapt the specifics
to this project:

```
[ ] Did I search for existing helpers before writing new ones?
[ ] Does every new write path stamp the required audit columns?
[ ] Does every new module have a logger?
[ ] Are all new business thresholds named constants?
[ ] Is every new function under this project's size guideline?
[ ] Does every new list endpoint use the shared pagination helper?
[ ] Did I verify the actual result (staged diff, real remote state, a
    reproduced fix) rather than trusting a tool's reported success?
    (See "Verify, Don't Assume" above.)
[ ] If this touches a Security Review Trigger file or a production
    surface: has an actual human reviewed the diff — not just CI, not
    just the author's own confidence?
```

## Security Review Trigger

Any change to these files should trigger a mandatory, deliberate security
review before merge — not just ordinary code review:

```
[List this project's actual auth/security/rate-limit/session files here
as soon as they exist.]
```

**Also add deployment-infrastructure config files to this list once they
exist** (compose files, database role/grant scripts, auth config for the
database itself) — real, severe bugs live in exactly these files more
often than intuition suggests, and they are easy to forget belong on
this list because they're config/SQL, not application code.

---

## Project-Specific Context for AI Sessions

```
[Fill in as the project takes shape:

Domain:      [what this project actually does]
Stack:       [languages, frameworks, database]
Auth:        [how authentication/authorization actually work]
Testing:     [test runner, whether the DB is mocked or real, how CI
              sets up its test environment]
Background jobs: [scheduler, queue, whatever applies]

Key design-decision records to read before touching shared
infrastructure: [list them here as they accumulate — see
docs/architecture/decisions/]
]
```

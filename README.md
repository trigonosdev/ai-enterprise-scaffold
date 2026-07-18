# AI Enterprise Scaffold

A starting point for building enterprise-grade software with heavy AI
assistance, without the specific failure mode that AI-assisted development
introduces: **context-reset drift** — the same helper reimplemented four
times, the same business rule encoded three slightly different ways, a
security guard nobody ever verified actually fires, because each
generating session had no memory of the sessions before it.

This is not a code framework. It doesn't install a package or scaffold an
app skeleton (`create-react-app`-style). It's a **discipline scaffold**: the
documents, structure, and process that keep an AI-assisted codebase
internally consistent as it grows past the size where any one person (or
any one AI session) can hold the whole thing in their head.

---

## What's in here, and why

| File / folder | Purpose |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | The core artifact. Loaded automatically by Claude Code (and readable by any AI tool) at the start of every session — the pre-generation checklist, architecture invariants, drift-prevention framework, and audit cadence. **This is what actually prevents drift**, not the folder structure around it. |
| [`AI_PROMPTS.md`](AI_PROMPTS.md) | A growing library of reusable, project-specific prompt templates, so each new task starts from a prompt that already encodes "search for existing patterns first," not a blank page. |
| [`docs/architecture/decisions/`](docs/architecture/decisions/) | Architecture Decision Records (ADRs) — a permanent, append-only log of *why* each significant technical decision was made. The single best defense against an AI session "helpfully" reverting a decision it doesn't have the context for. |
| [`docs/CASE_STUDIES.md`](docs/CASE_STUDIES.md) | Empty until this project has its first real drift incident. Then: write down what broke, why ordinary review missed it, and what question finally surfaced it. A real case study from *this* project is worth more than any amount of generic advice — including everything in this README. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Process and engineering conventions — the PR checklist, commit conventions, testing standards. |
| [`docs/CI_CD_PATTERNS.md`](docs/CI_CD_PATTERNS.md) | Two CI/CD patterns worth adopting once there's real code: a test-suite gate, and a release pattern that includes an actual deployment smoke test — not just "build succeeded." Shown as copy-paste examples, not live workflow files, so a fresh clone doesn't show a failing check with nothing real to test yet. |

---

## What this scaffold is *for*

Three concrete things it gives a new project on day one, instead of
discovering the need for them the hard way after month three:

1. **A place for "why" to live that isn't a Slack thread.** ADRs mean the
   next session — human or AI — can find out *why* something unusual-looking
   was built that way, instead of "helpfully" simplifying it back to the
   naive version.
2. **A pre-generation habit, not a post-hoc cleanup habit.** The checklist
   in `CLAUDE.md` is meant to run *before* code is written, forcing a
   search-for-existing-patterns step that no AI session does by default —
   catching drift before it's committed, rather than auditing it out later.
3. **A named cadence for the audit that per-PR review structurally cannot
   do.** Individual PRs look correct in isolation. Drift is only visible at
   the whole-codebase level, asking "does this whole subsystem still make
   sense" — a question nobody is naturally prompted to ask unless a
   recurring audit exists whose entire job is asking it.

## What this scaffold is *not*

- Not a guarantee against bugs. It's a structural defense against a
  *specific* failure mode (cross-session inconsistency), not a substitute
  for testing, review, or judgment.
- Not a one-time setup you do and forget. `CLAUDE.md`'s checklist, the ADR
  discipline, and the audit cadence only work if they're actually used on
  every session, every PR, every quarter — a scaffold nobody follows
  provides exactly as much protection as no scaffold at all.
- Not opinionated about language, framework, or domain. Every concrete
  example in `CLAUDE.md` is a placeholder — `[bracketed]` — meant to be
  replaced with this project's real stack and real conventions the first
  week, not left generic.

---

## How to start a new project from this scaffold

```bash
# 1. Clone this repo under a new name
git clone https://github.com/<your-username>/ai-enterprise-scaffold.git my-new-project
cd my-new-project

# 2. Point it at a new, empty remote (don't push to this scaffold's history)
git remote remove origin
git remote add origin https://github.com/<your-username>/my-new-project.git

# 3. Start a clean history — you want your new project's own commit log,
#    not this scaffold's
rm -rf .git
git init
git add -A
git commit -m "Initial commit from ai-enterprise-scaffold"
git branch -M main
git push -u origin main
```

Then, **before writing any application code**:

1. Open `CLAUDE.md` and replace every `[bracketed]` placeholder — project
   name, stack, layer rules, naming conventions, shared-helper locations
   (empty for now; you'll fill this in as the first few helpers get
   written), security-review-trigger file list (also starts empty).
2. Delete this README's content and replace it with your actual project's
   README — this file's job was to explain the scaffold, not describe your
   product.
3. Write ADR-0001 recording the decision to start from this scaffold and
   why (a real ADR template is already in
   `docs/architecture/decisions/template.md`) — this is also a good forcing
   function to confirm you've actually read `CLAUDE.md` before writing
   anything else.
4. If using Claude Code specifically: `CLAUDE.md` at the repo root is
   picked up automatically, every session, with no further setup.

## Keeping the scaffold current as the project grows

- Every time a genuinely shared helper gets written, add it to `CLAUDE.md`'s
  Shared Helpers section immediately — not "later," since "later" is how
  the first duplicate gets written.
- The first time something breaks *because* an AI session couldn't see
  prior context, write it up in `docs/CASE_STUDIES.md`. Don't wait for a
  second one to feel like it's worth documenting — the case study is more
  valuable the closer it is to when institutional memory of it is still
  fresh.
- Actually run the audit cadence in `CLAUDE.md`. Put it on a real calendar
  or a real milestone trigger, not "whenever someone remembers."

---

## License

[Choose a license for your new project — this scaffold itself is provided
as-is for reuse; replace this section once you've cloned it.]

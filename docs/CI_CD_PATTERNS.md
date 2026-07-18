# CI/CD Patterns

Two patterns worth adopting from day one, once this project has a real
stack to test. These are shown as illustrative examples, not live
workflow files — copy the pattern into `.github/workflows/` (or your
CI system's equivalent) once there's real code and a real deployment
mechanism for them to exercise; a placeholder workflow with nothing real
to test just produces a confusing failing check on every push.

---

## Pattern 1: Application tests, gated on PR

The ordinary test suite — unit tests, integration tests, linting, type
checking. Standard practice; included here mainly to note one thing
that's easy to get wrong: **the database/environment your test suite sets
up for itself is very often not the same environment your documented
production deployment sets up.** That gap is exactly where Pattern 2
below earns its keep — a green run of Pattern 1 alone does not prove
Pattern 2's claim.

```yaml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # [Set up your language/runtime here]
      # [Install dependencies]
      # [Run lint / type-check]
      # [Run the actual test suite]
```

## Pattern 2: Release with a real deployment smoke test

The harder-won pattern. A build that succeeds and a deployment that
actually works are different claims — the first is easy to verify
automatically and often is; the second is easy to *assume* is covered by
the first and often isn't, because nothing explicitly checks it.

The shape that closes that gap: on every release, actually stand up the
real, documented deployment mechanism — from a genuinely empty starting
state, using the exact artifact (image, package, whatever) that was just
published, not a locally-built approximation of it — and assert a real
health check before considering the release done.

```yaml
name: Release

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # [Build and publish your release artifact — container image,
      #  package, binary, whatever this project ships]

      # The step that actually matters:
      - name: Smoke test the real deployment path
        run: |
          # 1. Stand up the deployment using the artifact JUST published
          #    above, not a locally-built stand-in for it.
          # 2. From a genuinely fresh/empty starting state — a volume
          #    that's never been initialized before, not a reused one
          #    left over from a prior run.
          # 3. Wait for and assert a REAL health check response — not
          #    "the process started," an actual "is this thing correctly
          #    serving requests" check.
          # 4. Tear it down. If any step fails, the release fails —
          #    that's the point.
          echo "Replace this with your actual deployment mechanism."
```

### Why "genuinely fresh starting state" is the detail that matters most

A smoke test that reuses a volume/database/environment left over from a
previous successful run doesn't actually prove anything about a new
customer's first deployment — it just proves the *incremental* path
works, which is a meaningfully weaker claim. The bugs most worth finding
this way are exactly the ones that only show up once, on first
initialization: a role that only gets created on a truly empty database,
a config generation step that only runs when a file doesn't already
exist, a migration that assumes prior state that a fresh install
wouldn't have. Test from empty every time, or this pattern quietly stops
proving what it's supposed to prove.

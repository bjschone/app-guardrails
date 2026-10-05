# Large Tier

Up to ~10,000 users, significant client money ($10k+), built to last and to be
maintained by people who weren't in the room. Everything in Medium applies. Large adds
what "professional grade" actually means in practice: architecture that survives long AI
sessions, tested-not-hoped correctness, transaction safety, and handoff-grade docs.

Two risks define this tier. First, **iteration itself becomes a threat**: a peer-reviewed
study (Shukla et al., IEEE-ISTAS 2025) measured 37.6% more critical vulnerabilities after
five rounds of AI "improvement" with no human review between rounds. It tested one model
(GPT-4o) on C and Java, so treat the number as a warning, not a constant - but the
mechanism is easy to see: "improvements" quietly remove validation, relax types, and
widen scopes. Second, **the maintainer isn't you**: every shortcut that's fine when the
author holds the context becomes a trap for the next developer.

## Build

- **ARCHITECTURE.md before code.** Written and agreed before the first feature: modules
  and their responsibilities, data flow, where each kind of logic lives, key constraints
  ("all DB access goes through the repository layer," "no business logic in route
  handlers"). One to two pages. It's the contract every session builds against.
- **Start with a domain model, not endpoints.** In ARCHITECTURE.md, list the entities,
  their relationships, and the invariants that must always hold: "inventory ≥ 0," "an
  order always belongs to a customer," "a user has exactly one active subscription."
  Invariants are the cheapest correctness tool in the whole document - every one is a
  test, a validation rule, and a design constraint in a single sentence. AI skips
  modeling and goes straight to routes; don't let it.
- **Session discipline** - the countermeasure to context decay on long builds:
  - Start: re-read ARCHITECTURE.md, REGRESSIONS.md, and the recent decision log before
    writing code.
  - End: append a short summary to the decision log - what changed, what's half-done,
    what the next session needs to know.
  - Changes to ARCHITECTURE.md are deliberate, discussed with the user first, and logged -
    never silent drift.
- **Complexity as smoke, not gates.** Treat cyclomatic ~10 / cognitive ~15 per function
  as flags for review, not numbers to satisfy - chasing thresholds makes AI shred logic
  into fragments that are individually simple and collectively incomprehensible, which
  is its own failure mode. The real bar: refactor what's hard to reason about. Watch for
  god-modules: a file that keeps absorbing new functionality because it's already open
  is the monolith pattern in progress - split it when it takes on a second
  responsibility, not when it's 2,000 lines.
- **No duplication.** Blocks of 10+ lines appearing twice get extracted immediately.
  Duplicates are how one copy gets the security fix and the other keeps the hole.
- **Transaction safety.** Every multi-step state change (sequential DB writes, file
  operations plus a DB write, external API call then a local mutation) either completes
  fully or rolls back. If step N fails, steps 1 through N-1 get compensated. Partial
  states that "shouldn't happen" are the ones that page you.
- **Concurrency by design.** Concurrent writes to the same record use optimistic
  locking or a real transaction, and duplicate requests (double-click, client retry,
  webhook redelivery) are handled deliberately, not by luck. "Two users edited the same
  thing" is a normal Tuesday at 10,000 users.
- **Operational spine:** correlation IDs on every request, basic metrics (request rate,
  error rate, latency), and logs structured enough to follow one request through the
  whole system. Debugging an AI-generated codebase without this is archaeology.
- **Every cache has an invalidation story** - what makes an entry stale and what clears
  it, written down when the cache is added. A cache without one is a bug generator on a
  timer.
- **Risky changes ship behind a flag** with a kill switch, so rollback is a toggle
  rather than a redeploy.
- **Never silently break an API contract.** Anything external consumes gets versioned
  or the change gets coordinated - AI changes response shapes without announcing it,
  and the consumer finds out in production.
- **Type discipline.** Function signatures tell the truth: a function that can fail says
  so in its return type or its documented throws - never a typed success path with a
  secret undefined on error.

## Testing

- **80% behavioral coverage on new code** - behavioral meaning every test asserts a
  specific outcome. Coverage percentage from presence-only tests doesn't count.
- **Explicit edge tracing** on every function handling collections or external data:
  empty, null, single item, malformed. Trace them; don't assume them.
- Failure-path tests, not just happy-path: what does the API return when the DB is down,
  when auth fails, when input is garbage.
- Tests written to catch AI-generated code's real gaps - not AI-generated tests that
  mirror the code's own assumptions back at it. When a test only restates what the
  implementation does, it validates nothing.
- **Security regression tests.** Every security finding that gets fixed gets a test
  that fails if it comes back - the executable version of REGRESSIONS.md, aimed at
  iteration drift.
- **Fuzz or property-test anything that parses untrusted formats** - file imports,
  webhook payloads, query strings with structure. Parsers are where malformed input
  becomes remote code execution.
- **Load-test the hot paths before launch.** Know where it breaks before 10,000 users
  find it for you.

## Docs (handoff grade)

The test: a competent developer who has never spoken to you can orient, run, and safely
change the app.

- **Onboarding README**: what it is, local setup that actually works from scratch,
  deploy process, environment variables, where things live.
- **ARCHITECTURE.md** kept current - it describes the code as it is, not as it started.
- **Decision log** (from Medium) continued for the life of the project. Significant
  architectural decisions additionally get a lightweight ADR - context, decision,
  consequences, ten lines - so the next developer inherits the *why*, not just the what.
- **Runbook basics**: how to check health, where logs go, the three most likely failures
  and what to do about them.

## Audit

**Full Audit** - all six passes in `audits.md` - at major milestones and as a
**pre-delivery gate**. The gate is real: Critical or High findings block delivery.

Plus the Large-only standing rule: **any change to security-sensitive code (auth,
validation, crypto, session handling) triggers the regression pass immediately** - not
at the next milestone. Verify the change didn't remove or weaken an existing control:
the JWT check that lost its algorithm pin, the new query that regressed to string
interpolation in a file where everything else is parameterized. This is the direct
countermeasure to feedback-loop degradation, and it's the discipline that separates this
tier from the other two.

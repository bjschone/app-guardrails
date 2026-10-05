# Audits

Three audit levels, matched to tier. All three share one operating principle: **don't
trust the appearance of correctness.** AI-generated code optimizes for looking right -
it produces plausible structure, tidy comments, and error handling that's aesthetically
correct but semantically hollow. Audit by tracing actual execution paths and data flows.
Never mark a check passed because the code "clearly handles" something; follow the code
until you've seen it handle it.

Report findings using the severity table at the bottom. Critical findings block "done"
at every tier.

---

## Critical Sweep (Small - one pass, ~20 minutes)

The minimum civilized check before calling anything finished. Six hunts:

1. **Secrets.** Scan every source and config file (including examples and anything
   gitignored-but-committed) for API keys, tokens, connection strings, signing secrets
   as literals. Anything found is Critical.
2. **Injection surfaces.** Find every point where user input reaches a query, shell
   command, file path, rendered HTML, or an outbound request. Verify parameterization,
   safe APIs, or escaping at each one - and for any endpoint that fetches a
   user-supplied URL, verify destination allowlisting (SSRF). Check new code even when
   old code does it right - regression to concatenation is the pattern.
3. **Async error propagation.** For every async operation: does a handler exist, and
   does its catch block propagate a real signal (rethrow, typed fallback, central
   handler, error state)? Flag every catch that only logs and falls through - the caller
   is receiving undefined and doesn't know it.
4. **Auth and ownership** (if the app has accounts). Spot-check routes that access
   user-owned resources: is auth enforced server-side, and does the handler verify the
   requesting user owns *this specific resource*? Try the thought experiment: what
   happens if a logged-in user requests someone else's resource ID?
5. **Dead weight.** Modules nothing imports, functions nothing calls, state written but
   never read, guards for impossible conditions. Delete, don't comment out.
6. **Dependencies.** Every entry in the manifest exists on its official registry and is
   actually imported somewhere. Unfamiliar package names get verified, not trusted.

If the app has LLM features, add the five hunts from `llm.md`'s audit hook to this
sweep.

---

## Standard Audit (Medium - three passes)

Run at milestones and before delivery. Includes everything in the Critical Sweep,
distributed across the passes.

### Pass A - Architecture consistency

- **Pattern adherence.** Take the declared pattern (README/PATTERN.md) and hold every
  module against it. Deviating modules are usually late-session additions where context
  decayed - flag each one with what rule it breaks.
- **Abstraction audit.** For every interface or abstraction layer: would deleting it and
  using the concrete thing directly change any behavior? If no, it's cosmetic - flag for
  removal. An abstraction earns its place by hiding complexity, not relocating it.
- **Orphan hunt.** Modules with no callers; state initialized but never cleaned up or
  never read; near-duplicate functions in different files (the context-loss signature).
- **Convention drift.** Naming or style that shifts mid-file or between modules - each
  shift marks a session boundary worth extra scrutiny, because integration assumptions
  break there.

### Pass B - Async and state lifecycle

- **Handler coverage.** Every promise, await, and callback chain has error handling.
  List the ones that don't.
- **Propagation trace.** For every catch block: trace what the caller receives on
  failure. Acceptable: rethrow, typed fallback, central handler, surfaced error state.
  Unacceptable: log-and-undefined, empty catch, generic message swallowing the cause.
- **Teardown check.** Every listener, subscription, timer, and connection opened in a
  setup path has a matching teardown in the cleanup path. Missing teardowns are memory
  leaks and stale-state mutations waiting for load.
- **Race surface.** Find shared state written by more than one async path (double-fire
  handlers, polling loops without cancellation, message handlers mutating state without
  queuing). Verify serialization or locking; "it hasn't happened yet" is not a
  mechanism.
- **Boundary trace.** For each function processing collections or external responses:
  walk through empty, null, and single-item inputs by hand.

### Pass C - Security

- Secrets scan (Sweep item 1, full depth - include env example files, which are easy to
  fill with real values by accident).
- Injection surfaces (Sweep item 2, every entry point).
- **Authorization completeness.** Map every route/endpoint. For each: (a) server-side
  auth enforced? (b) resource-level ownership verified? (c) tokens validated for
  signature, expiry, and algorithm - not just presence? Client-side-only checks count as
  absent.
- **CORS and headers.** No wildcard origins on authenticated endpoints; CSP,
  X-Content-Type-Options, X-Frame-Options, HSTS present.
- **Session hygiene.** Cookies are HttpOnly, Secure, SameSite; sessions expire; tokens
  rotate on privilege change.
- **Upload handling** (if the app accepts files). Server-side MIME validation, size
  limits, random filenames, storage outside the web root.
- **Crypto spot-check.** Passwords hashed with a real KDF (bcrypt/Argon2/PBKDF2 - never
  MD5/SHA-1); tokens from a cryptographically secure source (never Math.random or
  timestamps); no hand-rolled encryption.
- **Log leakage.** No PII, tokens, credentials, or raw request bodies in log output; no
  stack traces or internal paths in HTTP responses.
- **Dependency check** (Sweep item 6) plus a pass for known-vulnerable versions, pinned
  versions, and a committed lockfile.
- **LLM surface** (if the app has LLM features). Run the five hunts in `llm.md`'s audit
  hook as part of this pass.

---

## Full Audit (Large - six passes)

Everything in the Standard Audit plus deeper passes. Sequence matters - each pass
surfaces what the previous ones can't.

### Pass 0 - Inventory (orientation before judgment)

- Map the topology: every module, what it exports, imports, and what calls it. Flag
  modules importing from 5+ sources (god-module candidates) and modules imported by 10+
  consumers (highest-priority audit targets - a defect here fans out).
- Estimate session boundaries from style shifts and commit history. Boundaries between
  AI sessions are the highest-probability sites for silent contract violations.

### Pass 1 - Architecture (Standard Pass A, full depth)

Run Pass A across every module, not a sample. Additionally trace dead code paths:
branches that can't be reached, return values nothing consumes, imports nothing
references.

### Pass 2 - Async and state (Standard Pass B, full depth)

Run Pass B exhaustively - every async operation inventoried, every catch traced.

### Pass 3 - Security (Standard Pass C, full depth)

Run Pass C against every route and every input path, not spot-checks.

### Pass 4 - Logic and business rules

Semantic errors - code that's syntactically fine and logically wrong:

- **Conditional exhaustiveness.** Conditions always true/false; branches ordered so an
  earlier condition shadows a later one; assignment where comparison was meant.
- **Return consistency.** Every code path in a function returns the same type. The
  typed-success-path-with-secret-undefined-on-error pattern gets flagged everywhere it
  appears.
- **Data flow integrity.** Follow each user-facing input from entry to storage/output:
  validation at the entry boundary, encoding at the exit boundary, no
  deserialize-reserialize steps that change meaning.
- **Atomicity.** Every multi-step state change rolls back on partial failure. Name the
  compensation for each step or flag its absence.

### Pass 5 - Quality and maintainability

- **Duplication scan.** 10+ line blocks appearing more than once - each is a future
  half-patched vulnerability.
- **Complexity flags.** Flag functions past cyclomatic ~10 / cognitive ~15 for review -
  more branches means more paths to test and more places for one to go untested - but judge by
  whether the function is hard to reason about, not by the number alone. Also flag the
  opposite smell: logic shredded into fragments to duck a threshold.
- **Test quality, not test count.** Classify tests: behavioral (assert specific
  outcomes), presence-only (just confirm no throw), or circular (AI-generated to mirror
  AI-generated code). Only behavioral tests count toward the 80% bar.
- **Config validation.** Startup verifies all required environment variables; no silent
  fallbacks that let the app half-run misconfigured.

### Pass 6 - Iterative regression (the AI-specific pass)

The pass that exists because iteration can degrade security (one IEEE-ISTAS 2025 study
measured 37.6% more critical vulnerabilities after five unreviewed AI refinement rounds -
see tier-large.md):

- **Before/after on security-sensitive changes.** For every modification to auth,
  validation, crypto, or session code: compare against the prior version. Did the
  change remove a check, relax a type, widen a scope, drop an algorithm pin? An
  "improvement" that weakens a control is a regression regardless of what it improved.
- **Surface-approximation trap.** For each security-critical block, verify completeness,
  not presence: JWT validation that checks signature but not algorithm or expiry;
  parameterization applied to new queries while old raw queries sit in the same file;
  validation added at one entry point while a second entry point bypasses it.
- **Session-boundary contracts.** At each boundary identified in Pass 0, verify the
  producer and consumer actually agree - value shapes, error semantics, null behavior.
  Stale assumptions across boundaries break silently.

---

## Severity and reporting

| Severity | Examples | Action |
|---|---|---|
| **Critical** | Hardcoded secret; injection; auth bypass; missing ownership check (IDOR); fabricated dependency | Blocks "done" - fix immediately, at every tier |
| **High** | Swallowed async error on a production path; wildcard CORS on authed endpoint; weak crypto; missing rollback on multi-step writes | Fix before delivery/release |
| **Medium** | Orphan state without guards; missing validation on a non-critical path; dead module; missing teardown | Fix within the current work block |
| **Low** | Naming drift; duplicate logic block; comment noise | Fix opportunistically |
| **Info** | Cosmetic abstraction; phantom guard; over-specified edge case | Note it; prune when touching that code |

Report every audit as: findings grouped by severity, each with file/location, what's
wrong, and the one-line fix direction. Then fix Criticals before saying anything is
finished. Log audit-found bugs in REGRESSIONS.md like any other bug.

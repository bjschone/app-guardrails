# Medium Tier

100-2,000 users, possibly a paying client, real accounts and user content. Everything in
Small still applies - especially the pruning instinct. Medium adds structure where scale
and money create consequences: consistent architecture, deliberate error handling,
behavioral tests, and performance basics.

The defining risk at Medium is **drift**. Projects this size take enough sessions that
context decay becomes real: patterns established early get abandoned late, near-duplicate
utilities appear, naming conventions mutate mid-file. The countermeasures below are
mostly about making early decisions durable.

## Build

- **Declare the architecture before coding.** One short section at the top of the README
  (or a PATTERN.md if it needs more room): the pattern in use (layered, MVC, repository -
  whatever fits), where each kind of logic lives, and how modules talk to each other.
  Five to ten sentences, not a document. **Re-read it at every session start** and hold
  every new module to it. A module that deviates is a bug even if it works.
- **Session-end drift check, 60 seconds:** did naming shift, did a near-duplicate
  function appear, did anything bypass the declared pattern or an existing abstraction?
  Catch drift while it's a rename, not a refactor.
- **One error-handling strategy, applied everywhere.** Decide once: how errors flow
  (central handler or per-boundary), what the error shape is, what users see vs. what
  gets logged. Classify errors by kind - validation, authorization, business rule,
  infrastructure, unexpected - so handling and messaging can differ by class instead of
  every failure producing the same generic message. Errors on the same layer look the
  same; asymmetric one-off handling is where swallowed errors hide.
- **Consistent naming and style across sessions.** When the current file's conventions
  differ from the codebase's, match the codebase, then flag the inconsistency - don't
  add a third style.
- **Performance basics, not performance theater:**
  - Rate limiting on public endpoints (auth endpoints especially).
  - Pagination on any list that grows with users - no unbounded queries.
  - Watch for N+1 query patterns when fetching related data.
  - Front-end: ux.md §11 is part of this bar - LCP under 2.5s, no layout shift,
    lazy-loading, optimistic UI.
  - Nothing beyond that without a measured reason.
- **CORS and security headers** configured deliberately: no wildcard origins on
  authenticated endpoints; set Content-Security-Policy, X-Content-Type-Options,
  X-Frame-Options, Strict-Transport-Security. Sessions expire; tokens rotate on
  privilege change.
- **Schema changes go through migrations.** Forward migration plus a rollback plan,
  idempotent where possible, and a backup before anything destructive (which is also a
  floor confirmation trigger). "Just edit the table" is how client data dies.
- **Payment and webhook handlers are idempotent.** A retry or double-fire must never
  double-charge or duplicate records - use idempotency keys or natural deduplication.
  Networks retry; design for it.
- **Time is UTC internally, ISO-8601 at the edges,** converted to local only at
  presentation. Never compare formatted date strings - the bug hides until a timezone
  boundary finds it.
- **Minimum operability:** a health endpoint, structured logs in one machine-parseable
  format, and automated backups for any data users would miss - with one restore
  actually tested before the backups are trusted. Untested backups are a hope, not a
  plan.
- **If the app accepts file uploads:** validate MIME type server-side, enforce size
  limits, generate random filenames, never trust the extension, and store files outside
  the web root. Uploads are the front door AI forgets to lock.
- **Apply security patches promptly** once compatibility is verified - a model suggests
  versions from its training data, which are already stale the day it ships.

## UI / UX

`references/ux.md` continues to apply in full. Medium adds the complete accessibility
standard: read `references/accessibility.md` and hold every UI change to WCAG 2.2 AA -
the POUR sections, whichever conditional sections the app triggers (forms, media, custom
widgets, tables/charts), the tooling requirements (axe, Lighthouse ≥95), and the
done-checklist. Its honesty rule stands: never claim something is accessible on the
strength of automated checks alone - say what was and wasn't actually tested.

## Testing

- **Behavioral tests on core flows** - tests that assert specific outcomes ("submitting
  valid signup creates a user and returns 201"), not tests that merely confirm functions
  run without throwing. A high test count with no real assertions is coverage theater.
- For every function that processes a collection, cover the three cases AI reliably
  misses: empty, null/missing, single item.
- Bug fixes get a regression test alongside the REGRESSIONS.md entry.

## Docs

- README: what it is, setup, deploy, environment variables (names and purpose, never
  values).
- **Decision log**: a running list of one-liners - date, decision, why. "Chose polling
  over websockets - simpler, 30s staleness is fine." Cheap to write, and it's the
  document that stops session five from relitigating session one.
- **Record assumptions alongside decisions:** "emails are unique," "inventory can't go
  negative," "one workspace per user." Unstated assumptions are where AI design errors
  hide - a written assumption can be challenged; an implicit one just breaks.

## Audit

**Standard Audit** (three passes, defined in `audits.md`): architecture consistency,
async/state lifecycle, security. Run at natural milestones - end of a feature block,
before showing a client - and always before delivery. Don't save auditing for the end;
drift found early is a rename, drift found late is a rewrite.

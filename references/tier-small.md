# Small Tier

Personal tools, prototypes, family projects, games. Fewer than 100 users, no money
changing hands, nothing sensitive stored. The floor applies in full; almost everything
else gets deliberately skipped.

Small's defining job is **pruning**. AI over-engineers by default - phantom guards for
impossible conditions, interfaces wrapping a single concrete type, service layers for
three functions, comments narrating trivial lines. At this size, that reflex is the main
threat to quality. The discipline isn't adding rigor; it's saying no.

## Build

- **Simplest structure that works.** One deploy unit. Flat file layout until it actually
  hurts - don't pre-build folder hierarchies for code that doesn't exist yet.
- **Minimal dependencies.** Before adding any package, ask whether 20 lines of plain code
  does the job. Each dependency needs a one-sentence justification; "it might be useful"
  isn't one.
- **No speculative abstraction.** No interfaces with one implementation, no factories, no
  config systems for values that never change. Extract an abstraction the second time
  something repeats, not the first time you imagine it might.
- **Handle edge cases that can occur, not edge cases that can't.** A guard for a
  condition the code path makes impossible is noise wearing a safety vest. When adding a
  check, be able to name the input that triggers it.
- **Comments explain why, never what.** Delete comments that narrate the line below them.
- **Refactor as you go.** If a file is turning into a junk drawer, split it now - linear
  growth without refactoring is the most common AI structural failure, and it starts
  small.

## UI / UX (if it has a UI)

`references/ux.md` applies in full - yes, even at this size. Small doesn't mean sloppy;
the complexity budget, the five states rule (empty, loading, partial, loaded, error),
and the 3-second cold test are exactly what keep a small app from drifting into clutter.
What Small skips is the full accessibility standard: the floor basics plus ux.md §9
cover it here - load `references/accessibility.md` only if the user asks.

## Testing

- No formal test suite. Instead: a **smoke-test checklist** in the README - the 5-10 core
  flows a human walks through before calling it done. Actually walk through them; the
  floor's "never fake it" rule covers claiming untested things work.
- Any bug found and fixed goes in REGRESSIONS.md, and its trigger gets added to the
  smoke-test checklist.

## Docs

- One README: what it is, how to run it, how to deploy it, the smoke-test checklist.
  That's it.

## Audit

- One **Critical Sweep** at the end of the build - see `audits.md`. About 20 minutes.
  Secrets, injection surfaces, async error propagation, auth/ownership if accounts
  exist, dead code, dependency check.

## Explicitly skipped at this size

Formal test suites, architecture docs, decision records, service layers, complexity
metrics, CI pipelines, rate limiting, structured logging. If the project starts drifting
toward Medium - a client appears, users multiply, sensitive data shows up - re-run
sizing rather than bolting on rigor ad hoc.

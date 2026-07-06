---
name: app-guardrails
description: Size and build apps with the right amount of rigor - no more, no less. Three t-shirt sizes (Small/Medium/Large) that scale testing, architecture, docs, and audit depth, on top of a non-negotiable security floor. Use this skill whenever the user starts building an app, web app, tool, prototype, game, or client project; asks to size or scope a build; requests a code audit, security review, or pre-delivery check; or mentions guardrails, best practices, or building something "right" or "solid" - even if they don't name a size.
---

# App Guardrails

Build apps at the right level of rigor. AI-generated code fails in predictable ways -
swallowed async errors, hardcoded secrets, missing ownership checks, architecture that
drifts as context fills. This skill prevents those failures during the build and catches
the rest in sized audits. It replaces guesswork about "how careful should I be here" with
one sizing decision made up front.

Two moves matter most: (1) size the project before writing any code, and (2) hold the
floor at every size. Everything else scales.

## Step 1: Size the project

Ask three questions before building. Don't skip this even when the user names a size - the
data-sensitivity question is the trap that catches "small" apps that aren't.

| Axis | Small | Medium | Large |
|---|---|---|---|
| **Audience** | <100 users | 100-2,000 | up to ~10,000 |
| **Stakes** | Personal, free, low-consequence | Paying client or real revenue | $10k+ contract or business-critical |
| **Data** | Nothing sensitive | Accounts, emails, user content | PII, payments, credentials, health/compliance data |

**Sizing rule:** the highest axis sets the size. One exception that keeps bureaucracy
out: when data sensitivity alone pushes the size up, apply the higher tier's *security
and audit* requirements but keep the lower tier's build/test/docs ceremony. A 20-user
tool that touches Stripe needs Large-tier security review - it doesn't need architecture
decision records.

Recommend a size, state which axis drove it, and let the user confirm or override.
Their override wins. If they invoked a size explicitly ("build this small"), skip the audience
and stakes questions but still confirm data sensitivity in one line.

Then read the matching tier file:

- **Small** → `references/tier-small.md`
- **Medium** → `references/tier-medium.md`
- **Large** → `references/tier-large.md`

Two more files load on their own triggers:

- **`references/ux.md`** - read it for any project with a user-facing UI, at every
  size. UX drift starts on day one; the complexity budget and five-states rule are what
  keep a Small app pleasant instead of merely functional.
- **`references/accessibility.md`** - the full WCAG 2.2 AA standard. Read it at Medium
  and Large, or at any size when the user asks. At Small, the floor's accessibility basics
  plus ux.md's accessibility floor section cover it.
- **`references/llm.md`** - read it whenever the app calls an LLM, runs agents or
  tools, or processes AI-generated content, at every size. Prompt injection is becoming
  the new SQL injection, and none of the classic guardrails cover it.

Audit instructions for all sizes live in `references/audits.md` - read it when an audit
pass comes due, not before.

## Step 2: Hold the floor (every size, no exceptions)

These are cheap to do during generation and expensive to retrofit. They apply to a
weekend puzzle game and a $10k client build alike. Size scales effort - it never scales
these down.

1. **Never fake it.** No fabricated data, no placeholder content presented as real, no
   claiming something works without running it, no stubbed functions that pretend to
   succeed. When reporting status, distinguish **verified** (ran it, saw it) from
   **inferred** (should work, based on reading the code) from **assumed** - blurring
   those three is exactly how fake completion happens. If something can't be built or
   verified, say so.
2. **Secrets from environment only - and they stay server-side.** No credential, API
   key, or signing secret as a string literal anywhere, including example and config
   files. Secrets never reach the browser, client bundles, logs, or container images.
   Use least-privilege credentials, and never point development at production
   credentials. Validate required environment variables at startup and fail loudly if
   missing; scattered `process.env.X` references with silent fallbacks are how apps
   ship half-configured.
3. **No user input concatenated into queries, commands, paths, or outbound requests.**
   Parameterized queries, safe APIs, or explicit escaping - every time, even when input
   "looks clean," even when the rest of the file already does it right. Regression to
   string interpolation in new code is a documented AI failure mode. The same rule
   covers SSRF: never fetch a user-supplied URL server-side without validating the
   destination against an allowlist - an endpoint that takes a URL and fetches it is a
   probe into your internal network.
4. **Authorization is designed, not assumed.** If the app has accounts, every endpoint
   answers four questions: who can call this, who owns this resource, what actions are
   allowed, and what happens when the check fails. AI reliably writes authentication and
   forgets authorization - the login exists, the ownership check doesn't. Missing
   resource-level checks (IDOR) are the single most common critical vulnerability in
   AI-generated code and invisible to casual review. All checks server-side; session
   cookies are `HttpOnly`, `Secure`, and `SameSite`.
5. **Async errors propagate a real signal.** Every async operation has a handler, and
   every catch block does one of: rethrow, return a typed fallback, call a central
   handler, or surface an error state. Catch-log-return-undefined is forbidden - it
   makes the caller crash somewhere else, later, mysteriously.
6. **Know your trust boundaries; validate at every one.** Name them before building:
   browser ↔ server, server ↔ database, server ↔ third-party APIs, anything ↔ LLM.
   Never trust data crossing a boundary - type, null, and range checks live at the
   boundary itself, not deep in the stack, not never. Most security bugs are a trust
   boundary someone forgot they had.
7. **Nothing sensitive in logs.** No PII, tokens, credentials, or request bodies in log
   output. Strip debug logging from production paths before calling anything done.
8. **Verify everything you import - packages and APIs alike.** Every dependency exists
   on its official registry before it's added (AI fabricates plausible package names;
   attackers register them), versions are pinned, and the lockfile is committed. Prefer
   few dependencies over many. The same skepticism applies to framework, SDK, and cloud
   APIs: when unsure a method exists, check the docs instead of trusting memory - AI
   invents method signatures as readily as package names.
9. **Confirm before consequences.** Stop and ask before: destructive actions (dropping
   tables, deleting files, force-pushing), schema changes to existing data, anything
   that costs money, and anything touching production.
10. **Keep the REGRESSIONS.md loop.** Log every real bug fixed - one line: symptom,
    cause, fix. Re-read it at the start of every session. AI repeats its own bugs across
    sessions; this file is the memory that stops it.
11. **Accessibility basics.** Semantic HTML, keyboard operability, visible focus,
    sufficient contrast, labeled inputs, alt text, no color-only signals, respect
    `prefers-reduced-motion`. Nearly free at generation time, miserable to retrofit.
    (Full spec: ux.md §9; full WCAG 2.2 AA standard: accessibility.md at Medium+.)

## Step 3: Build with the tier's discipline

The tier file defines build behavior, testing depth, docs, and audit cadence. Two habits
apply at every size because they counter context decay - the documented root cause of
architecture drift in long AI sessions:

- **Session start:** re-read REGRESSIONS.md, plus whatever orientation docs the tier
  requires (Medium: the declared pattern; Large: ARCHITECTURE.md and the decision log).
- **Resist the default instincts.** Left alone, AI over-comments, over-guards against
  impossible edge cases, adds abstractions that hide nothing, and grows files instead of
  refactoring. Every tier prunes these; Small prunes hardest.

## Step 4: Audit on the tier's cadence

| Size | Audit | When |
|---|---|---|
| Small | Critical Sweep (~20 min, one pass) | End of build |
| Medium | Standard Audit (3 passes) | At milestones + before delivery |
| Large | Full Audit (6 passes) | At milestones + pre-delivery gate |

Read `references/audits.md` when a pass is due. Report findings by severity; anything
Critical blocks "done" - fix it before presenting the work as complete.

One principle governs all auditing: **don't trust the appearance of correctness.**
AI-generated code optimizes for looking right. Trace execution paths and data flows;
never grade on surface compliance.

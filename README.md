# App Guardrails

A Claude skill for building apps at the right level of rigor - no more, no less. It replaces guesswork about "how careful should I be here" with one sizing decision made up front, then holds a non-negotiable security floor at every size.

AI-generated code fails in predictable ways: swallowed async errors, hardcoded secrets, missing ownership checks, architecture that drifts as context fills. This skill prevents those failures during the build and catches the rest in sized audits.

## When it triggers

The skill activates when you start building an app, web app, tool, prototype, game, or client project; ask to size or scope a build; request a code audit, security review, or pre-delivery check; or mention guardrails or best practices - even without naming a size.

## How it works

Two moves matter most: size the project before writing any code, and hold the floor at every size. Everything else scales.

### 1. Size the project

Three questions set the tier. The highest axis wins.

| Axis | Small | Medium | Large |
|---|---|---|---|
| **Audience** | <100 users | 100-2,000 | up to ~10,000 |
| **Stakes** | Personal, free, low-consequence | Paying client or real revenue | $10k+ contract or business-critical |
| **Data** | Nothing sensitive | Accounts, emails, user content | PII, payments, credentials, health/compliance data |

One exception keeps bureaucracy out: when data sensitivity alone pushes the size up, apply the higher tier's security and audit requirements but keep the lower tier's build, test, and docs ceremony. A 20-user tool that touches Stripe needs Large-tier security review - it doesn't need architecture decision records.

### 2. Hold the floor

Eleven requirements apply to a weekend puzzle game and a $10k client build alike. Size scales effort - it never scales these down.

1. **Never fake it.** No fabricated data, no placeholder content presented as real, no claiming something works without running it. Distinguish verified from inferred from assumed.
2. **Secrets from environment only, server-side only.** No credentials as string literals anywhere. Validate required environment variables at startup and fail loudly if missing.
3. **No user input concatenated into queries, commands, paths, or outbound requests.** Parameterized queries, safe APIs, or explicit escaping - every time. Covers SSRF via an allowlist on server-side fetches.
4. **Authorization is designed, not assumed.** Every endpoint answers who can call it, who owns the resource, what actions are allowed, and what happens when the check fails. Missing resource-level checks (IDOR) are the most common critical vulnerability in AI-generated code.
5. **Async errors propagate a real signal.** Every catch block rethrows, returns a typed fallback, calls a central handler, or surfaces an error state. Catch-log-return-undefined is forbidden.
6. **Know your trust boundaries; validate at every one.** Type, null, and range checks live at the boundary itself.
7. **Nothing sensitive in logs.** No PII, tokens, credentials, or request bodies in log output.
8. **Verify everything you import.** Every dependency exists on its official registry, versions are pinned, the lockfile is committed. The same skepticism applies to framework and SDK method signatures.
9. **Confirm before consequences.** Stop and ask before destructive actions, schema changes to existing data, anything that costs money, and anything touching production.
10. **Keep the REGRESSIONS.md loop.** Log every real bug fixed in one line, and re-read it at the start of every session.
11. **Accessibility basics.** Semantic HTML, keyboard operability, visible focus, sufficient contrast, labeled inputs, alt text, no color-only signals, respect `prefers-reduced-motion`.

### 3. Build with the tier's discipline

Each tier file defines build behavior, testing depth, docs, and audit cadence. Two habits apply at every size to counter context decay: re-read orientation docs at session start, and resist the default AI instincts to over-comment, over-guard against impossible edge cases, add abstractions that hide nothing, and grow files instead of refactoring.

### 4. Audit on the tier's cadence

| Size | Audit | When |
|---|---|---|
| Small | Critical Sweep (~20 min, one pass) | End of build |
| Medium | Standard Audit (3 passes) | At milestones + before delivery |
| Large | Full Audit (6 passes) | At milestones + pre-delivery gate |

Anything Critical blocks "done." One principle governs all auditing: don't trust the appearance of correctness. Trace execution paths and data flows rather than grading on surface compliance.

## Repository structure

```
app-guardrails/
├── SKILL.md                        # Entry point: sizing, the floor, build and audit flow
└── references/
    ├── tier-small.md               # Small-tier build, test, docs, audit
    ├── tier-medium.md              # Medium-tier build, test, docs, audit
    ├── tier-large.md               # Large-tier build, test, docs, audit
    ├── ux.md                       # UX guidance for any user-facing UI, every size
    ├── accessibility.md            # Full WCAG 2.2 AA standard (Medium+ or on request)
    ├── llm.md                      # Guardrails for apps that call an LLM or run agents/tools
    └── audits.md                   # Audit instructions for all sizes
```

Reference files load on their own triggers: `ux.md` for any user-facing UI, `accessibility.md` at Medium and Large or on request, `llm.md` whenever the app calls an LLM or processes AI-generated content, and `audits.md` when an audit pass comes due.

## Usage

Install as a Claude skill by placing the repo where your skills are loaded, then invoke it at the start of any build or when an audit is due. The skill recommends a size, names the axis that drove it, and waits for confirmation before proceeding.

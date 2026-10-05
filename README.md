# App Guardrails

A Claude skill for building apps at the right level of rigor - no more, no less.

You size the project once, before any code gets written. Then the skill holds a security floor that never drops, whatever the size. A weekend game gets a 20-minute audit. A client build with payments gets six audit passes. Both get the same eleven non-negotiables.

I built this because AI-generated code fails in predictable ways: swallowed async errors, hardcoded secrets, missing ownership checks, packages that don't exist, and architecture that drifts as a long session wears on. I've used it on every project I've built with Claude since. It hasn't let me down, and it means every project starts with good bones.

## Install

### Claude Code (plugin)

```
/plugin marketplace add bjschone/app-guardrails
/plugin install app-guardrails@bjschone
```

### Claude Code (personal skill)

Clone it into your skills folder:

```
git clone https://github.com/bjschone/app-guardrails ~/.claude/skills/app-guardrails
```

Pull to update. To scope it to one project, clone into that project's `.claude/skills/` instead.

### Claude apps (web and desktop)

Download the ZIP from the [latest release](https://github.com/bjschone/app-guardrails/releases) and upload it in Claude's skills settings.

## Usage

You don't need to call it by name. It triggers when you:

- **Start a build** - "Let's build a habit tracker for my family." Claude asks three sizing questions, recommends a size, names the axis that drove it, and waits for you to confirm.
- **Name a size** - "Build this small." Claude skips audience and stakes but still asks whether any sensitive data is involved.
- **Ask for a check** - "Audit this before I send it to the client." Claude runs the audit that matches the project's size and reports findings by severity.
- **Come back to a project** - Claude re-reads `REGRESSIONS.md` and the project's orientation docs before writing code.

You can also invoke it directly with `/app-guardrails` in Claude Code.

## How it works

### 1. Size the project

Three questions set the tier. The highest axis wins.

| Axis | Small | Medium | Large |
|---|---|---|---|
| **Audience** | <100 users | 100-2,000 | up to ~10,000 |
| **Stakes** | Personal, free, low-consequence | Paying client or real revenue | $10k+ contract or business-critical |
| **Data** | Nothing sensitive | Accounts, emails, user content | PII, payments, credentials, health/compliance data |

One exception keeps the bureaucracy out. When data sensitivity alone pushes the size up, you get the higher tier's security and audit requirements but keep the lower tier's build, test, and docs ceremony. A 20-user tool that touches Stripe needs Large-tier security review. It doesn't need architecture decision records.

### 2. Hold the floor

Eleven requirements apply at every size:

1. **Never fake it** - and keep verified, inferred, and assumed separate when reporting status.
2. **Secrets from environment only, server-side only.**
3. **No user input concatenated into queries, commands, paths, or outbound requests.**
4. **Authorization is designed, not assumed** - every endpoint knows who owns the resource.
5. **Async errors propagate a real signal** - no catch-log-return-undefined.
6. **Know your trust boundaries; validate at every one.**
7. **Nothing sensitive in logs.**
8. **Verify everything you import** - packages and API methods alike.
9. **Confirm before consequences** - destructive actions, schema changes, money, production.
10. **Keep the REGRESSIONS.md loop** - one line per real bug fixed, re-read every session.
11. **Accessibility basics.**

Size scales effort. It never scales these down.

### 3. Build with the tier's discipline

Each tier file sets build behavior, testing depth, docs, and audit cadence. Small's main job is pruning what AI over-builds by default. Medium's job is stopping drift across sessions. Large's job is making the codebase safe for someone who wasn't in the room.

### 4. Audit on the tier's cadence

| Size | Audit | When |
|---|---|---|
| Small | Critical Sweep (~20 min, one pass) | End of build |
| Medium | Standard Audit (3 passes) | At milestones + before delivery |
| Large | Full Audit (6 passes) | At milestones + pre-delivery gate |

Anything Critical blocks "done." Every audit traces real execution paths instead of grading code on how correct it looks.

## What's in the repo

```
app-guardrails/
├── SKILL.md                # Entry point: sizing, the floor, build and audit flow
├── references/
│   ├── tier-small.md       # Small-tier build, test, docs, audit
│   ├── tier-medium.md      # Medium-tier build, test, docs, audit
│   ├── tier-large.md       # Large-tier build, test, docs, audit
│   ├── ux.md               # UX rules for any user-facing UI, every size
│   ├── accessibility.md    # Full WCAG 2.2 AA standard (Medium+ or on request)
│   ├── llm.md              # Guardrails for apps that call an LLM or run agents
│   └── audits.md           # Audit instructions for all sizes
└── .claude-plugin/
    └── marketplace.json    # Lets Claude Code install it as a plugin
```

Claude only loads what the moment needs. `SKILL.md` loads when the skill triggers, and the reference files load on their own triggers: the matching tier file after sizing, `ux.md` for any UI, `llm.md` for any LLM feature, `audits.md` when an audit comes due.

## Make it yours

Fork it and change the thresholds. If your "Small" is 500 internal users, or your Large tier needs SOC 2 evidence, edit the tier files. The structure - size first, hold the floor, scale the rest - is the part worth keeping.

## Sources

The skill cites research where it makes a factual claim:

- [OWASP Top 10:2025](https://owasp.org/Top10/) - broken access control is #1; "Mishandling of Exceptional Conditions" is new
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/) - prompt injection is LLM01
- [Veracode 2025 GenAI Code Security Report](https://www.veracode.com/press-release/ai-generated-code-poses-major-security-risks-in-nearly-half-of-all-development-tasks-veracode-research-reveals/) - insecure choices in 45% of tasks across 100+ LLMs
- [Spracklen et al., USENIX Security 2025](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen) - package hallucination rates
- [Shukla et al., IEEE-ISTAS 2025](https://arxiv.org/abs/2506.11022) - security degradation across iterative AI refinement
- [Chroma, "Context Rot" (2025)](https://www.trychroma.com/research/context-rot) - performance changes as input length grows
- [GitClear, AI Copilot Code Quality (2025)](https://www.gitclear.com/ai_assistant_code_quality_2025_research) - copy/paste vs. refactoring trends
- [Deque automated testing study (2021)](https://www.deque.com/blog/automated-testing-study-identifies-57-percent-of-digital-accessibility-issues) - automated accessibility coverage
- [WebAIM Screen Reader User Survey #10](https://webaim.org/projects/screenreadersurvey10/) - screen reader usage

## License

[MIT](LICENSE). Use it, change it, ship it.

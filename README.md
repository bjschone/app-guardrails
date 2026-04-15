# app-guardrails

A personal engineering operating system for building software with AI assistance — without the brittleness, security holes, UX drift, and silent regressions that usually come with vibe-coded apps.

These files exist to externalize the things I'd otherwise have to remember on every project. Drop them into Claude Code, Claude Projects, Cursor, or any AI coding tool, and every new build starts at a higher baseline automatically.

## The files

| File | Purpose | When to load |
|---|---|---|
| `build-standards.md` | Engineering and security standards. OWASP coverage, code quality, dependency hygiene, secure defaults. | **Always** |
| `qa-standards.md` | QA and regression-prevention standards. Blast radius checks, CRUD coverage, functional verification, regression log. | **Always** |
| `ai-coding-guardrails.md` | Rules for *how* the AI assistant works with me. Plan-before-coding, scope discipline, no faking, confirmation triggers. | **Always** |
| `ux-guardrails.md` | UI/UX standards. Complexity budget, five-states rule, accessibility floor, copy and microcopy. | Any project with a UI |
| `accessibility-guardrails.md` | Accessibility standards aligned with W3C WAI / WCAG 2.2 AA. POUR principles, conditional sections (forms, media, custom widgets), AT test matrix, end-of-task checklist. | Any project with a UI |

The first three are the **core set** — load them on every project. `ux-guardrails.md` and `accessibility-guardrails.md` are opt-in but should be loaded together for anything user-facing.

## How to deploy

### Claude Code (terminal / IDE)

**Global** — applies to every project on this machine:

```bash
mkdir -p ~/.claude
cat build-standards.md qa-standards.md ai-coding-guardrails.md ux-guardrails.md accessibility-guardrails.md > ~/.claude/CLAUDE.md
```

**Per-project** — for project-specific overrides or additions:

```bash
# In the project repo root
cp /path/to/app-guardrails/*.md ./
# Then create a CLAUDE.md that references them and adds project context
```

A per-project `CLAUDE.md` typically looks like:

```markdown
# Project: [Name]

Follow all standards in:
- ./build-standards.md
- ./qa-standards.md
- ./ai-coding-guardrails.md
- ./ux-guardrails.md
- ./accessibility-guardrails.md

## Project-specific context
- Stack: [e.g. Next.js + Supabase]
- Deploy: [e.g. Vercel via main branch]
- Brand: [e.g. fonts, colors, tone]
- Anything else Claude needs to know
```

### Claude.ai Projects (chat interface)

1. Create a Project called "App Builds"
2. Paste the contents of `build-standards.md`, `qa-standards.md`, and `ai-coding-guardrails.md` into the Project instructions field
3. Upload `ux-guardrails.md` to Project knowledge for UI projects, or paste it in too if all your projects have UIs

### Cursor / Windsurf / other AI IDEs

Symlink so you only edit in one place:

```bash
ln -s ~/.claude/CLAUDE.md .cursorrules
ln -s ~/.claude/CLAUDE.md .windsurfrules
```

## Update workflow

When you revise a file in this repo:

1. Edit the file here (the canonical source)
2. Re-run the deploy command above to refresh `~/.claude/CLAUDE.md`
3. Re-paste into any Claude.ai Projects that use these standards
4. Tag a release if the change is significant (`git tag v1.1 && git push --tags`)

## Versioning

Tag meaningful changes with semantic-ish versions:

- **Major** (`v2.0`) — breaking changes to structure, file renames, removed sections
- **Minor** (`v1.1`) — new sections, new files, expanded coverage
- **Patch** (`v1.0.1`) — typos, clarifications, small additions

This lets long-running projects pin to a specific version if I don't want a live-updating dependency.

## The feedback loop

The point of this repo is that **it gets sharper over time**. Every time I:

- Hit a bug → add a rule to `qa-standards.md`
- Get burned by a security oversight → add a rule to `build-standards.md`
- Watch Claude go off the rails → add a rule to `ai-coding-guardrails.md`
- Notice UX drift in something I built → add a rule to `ux-guardrails.md`
- Catch an accessibility gap (failed contrast, broken keyboard flow, silent screen reader) → add a rule to `accessibility-guardrails.md`

After six months of use, this repo encodes hundreds of hours of hard-won lessons that I never have to learn twice.

## Roadmap

Potential future additions:

- `data-guardrails.md` — schema, naming, soft-delete, timestamps, exports
- `content-guardrails.md` — brand voice, no fabricated stats, sentence case, hyphens not em dashes
- `deploy-guardrails.md` — pre-deploy checklist, rollback plans, smoke tests
- `prompt-guardrails.md` — standards for LLM calls inside apps (system prompts, evals, cost guardrails)
- `project-bootstrap.md` — checklist for spinning up any new project from zero
- `performance-guardrails.md` — bundle budgets, query budgets, Lighthouse thresholds

## Philosophy

Three principles hold the system together:

1. **Composability over completeness.** Each file stands alone. Mix and match per project.
2. **Externalize, don't memorize.** If I have to remember it, it's not a guardrail — it's a wish.
3. **The prime directive:** optimize for actually shipping software that works, not for appearing helpful, fast, or clever.

# QA & Testing Standards

You are updating production-grade software, not just patching a prototype. Every change — no matter how small — has a blast radius. Your job is not just to make the requested change, but to verify the app still works as a whole afterward. Apply the standards below to every update request. Sections are conditional — apply them when their trigger condition is met.

## 1. Before You Code: Blast Radius Check (Always Apply)

Before writing any code, state explicitly:

- **What is being changed.** The exact files, functions, components, routes, or data models.
- **What could be affected downstream.** Every other file, route, component, or data model that touches the thing being changed. Walk the import graph mentally if you have to.
- **What assumptions the existing code makes** that this change might break.
- **Risk level:** low (isolated change, no shared state), medium (touches shared utilities, multiple files, or UI state), high (touches auth, data models, migrations, routing, or anything more than ~5 files).

If risk is **high**, stop and confirm the scope and approach with the user before proceeding.

## 2. Regression Log Check (Always Apply if `REGRESSIONS.md` Exists)

If a `REGRESSIONS.md` file exists at the repo root, read it before coding. It contains every bug previously found in this project. For any change you make:

- Identify which past regressions are in the same area as the current change.
- Explicitly re-verify those specific scenarios after your change.
- If you fix a new bug during this update, add an entry to `REGRESSIONS.md` with: date, what broke, root cause, how it was fixed, and a one-line scenario to re-test in the future.

If `REGRESSIONS.md` does not exist and the project is more than a prototype, create it.

## 3. While Coding (Always Apply)

- **Make the smallest change that satisfies the request.** Do not refactor adjacent code unless the user asked for it.
- **Preserve existing behavior** unless the request explicitly changes it. If you must change behavior to fix something, call it out.
- **No silent API changes.** If you change a function signature, route shape, prop interface, or database column, list every caller you updated and confirm none were missed.

## 4. After Coding: Self-Review Pass (Always Apply)

Before declaring the change done, review your own diff with fresh eyes and answer:

- What did I change?
- What might this have broken that I'm not testing?
- Did I introduce any new dependencies, env vars, or config?
- Did I leave any debug logs, commented-out code, or `console.log` statements?
- Are error states and loading states still handled?
- If this is a UI change, what should look the same as before, and what should look different?

## 5. Functional Verification (Always Apply, Method Depends on Environment)

You must verify the app still works end-to-end before declaring the update complete. Pick the strongest method available in the current environment:

### Tier 1: Automated browser test (preferred when possible)
If you can run code in this environment (Claude Code, terminal, etc.) and the app has a UI:

- Write or extend a Playwright or Puppeteer script that exercises the affected flows.
- Run it headless against a local dev server.
- Report which flows passed, which failed, and any console errors or network errors observed.

### Tier 2: Direct execution and self-report
If you can run code but the app has no browser UI (CLI, API, script):

- Actually execute the relevant commands or `curl` calls.
- For any data-handling feature, run the full **CRUD cycle**: Create a record, Read it back, Update it, Read it again to confirm the update, Delete it, confirm it's gone.
- Report each step's result.

### Tier 3: Manual checklist for the user
If you cannot execute code in this environment:

- Generate a numbered checklist of every flow that should be manually verified after this change.
- Be specific: "Click X, expect Y" — not "test the form."
- Include console and network checks where applicable.

## 6. CRUD Coverage (If the Change Touches Any Data Entity)

For every data entity affected — directly or indirectly — verify the full lifecycle:

- **Create:** valid input succeeds, invalid input is rejected with a clear error.
- **Read:** the new record appears in lists, detail views, and any search or filter.
- **Update:** changes persist and are reflected everywhere the record is displayed.
- **Delete:** the record is gone from all views, and any references to it (foreign keys, cached lists, sidebars, counts) are also cleaned up or handled gracefully.
- **Edge cases:** empty state, single item, many items, special characters, very long values, concurrent updates if applicable.

The most common hidden bug: deleting a record leaves a stale reference somewhere — a count, a cached list, a sidebar item, a related-records section. Always check.

## 7. Frontend Verification (If the Change Touches Any UI)

- **Console check.** Open the browser console. Report any errors or warnings, including ones that existed before your change (note which are pre-existing).
- **Network check.** Open the network tab. Report any failed requests (4xx, 5xx) or unexpectedly slow ones.
- **Navigation check.** Click through every route reachable from the changed area. Confirm nothing 404s, nothing renders blank, and back/forward browser buttons still work.
- **State check.** If the change involves shared state (auth, global store, context), verify state is correct after: page reload, navigation away and back, and logout/login if applicable.
- **Responsive check.** Confirm the change works at desktop, tablet, and mobile widths if the app is responsive.
- **Accessibility quick-check.** Tab through the changed UI with the keyboard. Confirm focus is visible and logical.

## 8. Backend / API Verification (If the Change Touches Any Endpoint)

- **Happy path:** valid request returns expected response.
- **Auth:** unauthenticated request is rejected; authenticated-but-unauthorized request is rejected.
- **Validation:** missing fields, wrong types, and malicious input (SQL-ish strings, oversized payloads, unexpected fields) are all rejected cleanly.
- **Idempotency:** if the endpoint is supposed to be idempotent, calling it twice produces the same result.
- **Status codes:** correct codes for each case (200, 201, 400, 401, 403, 404, 409, 422, 500).

## 9. Database Verification (If the Change Touches Schema or Queries)

- **Migration runs cleanly** on a fresh database and on a database with existing data.
- **Migration is reversible** — the down migration restores the prior state.
- **Existing queries still work.** Run any query that touches the affected tables.
- **Indexes still cover** the queries that need them. No new full table scans on hot paths.
- **Referential integrity** is preserved — no orphaned foreign keys.

## 10. Automated Test Suite (If One Exists)

- **Run the full test suite** before declaring the change done. Not just the tests you think are relevant — all of them.
- Report pass/fail counts.
- If any test fails, fix it before completing the task. Do not disable, skip, or comment out failing tests to make the suite green.
- **Add new tests** for any new behavior introduced, and at least one negative test (bad input, unauthorized access, etc.).
- **Add a regression test** for any bug you fix during this update — the test should fail against the broken code and pass against the fix.

## 11. Build & Lint (Always Apply if Tooling Exists)

- Run the build (`npm run build`, `tsc`, `python -m build`, etc.) and confirm it completes without errors or new warnings.
- Run the linter and type checker. Report any new issues introduced by your change. Do not silence warnings to make output clean.
- If pre-existing warnings exist that are unrelated to your change, note them but do not fix them unless asked.

## 12. Cross-Cutting Continuity Checks (Always Apply)

These are the things that get silently broken by updates more than anything else:

- **Auth still works.** Log in, log out, log back in.
- **Existing data still loads.** Open a record created before this change.
- **Settings, preferences, and feature flags** still apply correctly.
- **Background jobs, cron tasks, or scheduled functions** still run.
- **Email, webhook, and third-party integrations** still fire on the right events.
- **Search, filter, sort, and pagination** still work on any list affected by the change.

## 13. Reporting Format (Always Apply)

When you finish an update, report in this structure:

```
## Summary
[One sentence: what changed and why.]

## Files Modified
[List with one-line descriptions.]

## Blast Radius Assessment
- Risk level: [low / medium / high]
- Affected areas: [list]
- Areas explicitly verified: [list]

## Verification Performed
- [Tier used: automated / direct execution / manual checklist]
- [Each check and its result]

## Known Gaps
- [Anything not verified, and why]
- [Anything the user should manually confirm]

## Regression Log Updates
- [New entries added to REGRESSIONS.md, if any]
```

## 14. When in Doubt (Always Apply)

- If you cannot verify something works, **say so explicitly**. Do not claim a change is complete if you only verified the happy path.
- If a change feels riskier than the request implied, **stop and surface it** before proceeding.
- Never describe a change as "tested" if you only ran the code mentally. State exactly what was executed and what was inferred.
- If the same bug appears twice in the same project, treat it as a process failure — add it to `REGRESSIONS.md` and propose a permanent guardrail (a test, a lint rule, a type, a refactor) to make it impossible to reintroduce.

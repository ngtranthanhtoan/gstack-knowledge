# 10 — Ship: The Final Mile

> A lot of branches die when the interesting work is done and only the boring release work is left. Humans procrastinate that part. AI should not.

Shipping is the phase where most AI-era sprints quietly fall apart. The feature is built, the tests are green, the review passed — and then nothing merges for three days because no one wants to do the release checklist. By the time you come back to it, you've forgotten what was in the diff.

This chapter is about treating the final mile the same way you treat the first mile: a disciplined process with artifacts that chain to the next thing.

## The phases of shipping

Shipping is actually five sub-phases, each with its own specialist:

```
/ship              →  branch → PR
/land-and-deploy   →  PR → deployed to production
/canary            →  production → verified-not-broken
/benchmark         →  production → performance baseline
/document-release  →  ship → docs that match reality
```

Run them in order. Each produces an artifact the next reads.

---

## `/ship` — branch to PR

**Specialist:** Release engineer.
**Job:** Once you've decided what to build, nailed the plan, and run a serious review, stop behaving like a brainstorm partner. Sync with main, run the right tests, make sure branch state is sane, update version/changelog, push, create or update the PR.

`/ship` is for a **ready branch**, not for deciding what to build. If you're still making scope decisions, you're not ready to ship.

### What it does, step by step

```
1. Sync with base branch (merge or rebase)
2. Bootstrap tests if missing (see below)
3. Run full test suite. Must be green.
4. Coverage audit — subagent runs independently, returns JSON:
   { coverage_pct, gaps, diagram, tests_added }
5. Plan completion audit — subagent checks shipped items against plan:
   { total_items, done, changed, deferred, summary }
6. Pre-landing review — same checklist as /review:
   SQL safety, LLM boundaries, race conditions, completeness gaps
7. Greptile comment triage (if enabled) — valid / already-fixed /
   false positive / suppressed
8. Version decision — see below
9. CHANGELOG entry — branch-scoped (see below)
10. Bisectable commits — infrastructure → models → controllers →
    VERSION/CHANGELOG/TODOS
11. Verification gate — if code changed after step 3, re-run tests
    before push (not "I'm confident")
12. Push and open/update PR
13. Conditional: run eval suites if prompt-related files changed
```

### Test bootstrap

If the project doesn't have a test framework, `/ship` sets one up:

- Detects the runtime.
- Researches the best framework for that runtime.
- Installs it.
- Writes 3-5 **real** tests for your actual code, not toy examples.
- Sets up CI/CD (GitHub Actions).
- Creates `TESTING.md`.

> 100% test coverage is the goal. Tests make vibe coding safe instead of yolo coding.

The test bootstrap is a one-time cost. After it runs once, every future `/ship` just runs tests.

### Coverage audit

Every `/ship` run builds a code path map from the diff, searches for corresponding tests, and produces an ASCII coverage diagram with quality stars.

Gaps get tests auto-generated. The PR body shows the coverage delta:

```
Tests: 42 → 47 (+5 new)
```

This is the single most useful field in a PR body. Reviewers glance at it, see the test count moved in the right direction, and move on.

### Review gate

`/ship` checks the **Review Readiness Dashboard** before creating the PR:

```
+====================================================================+
|                    REVIEW READINESS DASHBOARD                       |
+====================================================================+
| Review          | Runs | Last Run            | Status    | Required |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  1   | 2026-03-16 15:00    | CLEAR     | YES      |
| CEO Review      |  1   | 2026-03-16 14:30    | CLEAR     | no       |
| Design Review   |  0   | —                   | —         | no       |
+--------------------------------------------------------------------+
| VERDICT: CLEARED — Eng Review passed                                |
+====================================================================+
```

Eng Review is the only required gate. If it's missing, `/ship` asks — but won't block you. Decisions are saved per-branch so you're never re-asked.

### The version decision

Strict policy:

- **MICRO** (< 50 lines, trivial): no user confirmation.
- **PATCH** (50+ lines, no user-visible features): no user confirmation.
- **MINOR / MAJOR** (user-visible features or breaking changes): ask the user.

The point is to avoid both extremes: don't ask about every version bump (tedious), and don't silently bump MAJOR (catastrophic).

### CHANGELOG is branch-scoped

This is subtle but important: every feature branch that ships gets its own version bump and CHANGELOG entry. The entry describes what **this branch** adds — not what was already on main.

Key rules:

- Write the CHANGELOG entry at **ship time**, not during development.
- The entry covers ALL commits on this branch vs the base.
- **Never fold new work into an existing CHANGELOG entry** from a prior version that already landed. If main has v0.10.0 and your branch adds features, bump to v0.10.1 with a new entry — don't edit v0.10.0.
- After merging main, check: does CHANGELOG have your branch's own entry separate from main's? Is VERSION higher than main's? Is your entry topmost?

CHANGELOG is **for users, not contributors**:

- Lead with what the user can now *do* that they couldn't before.
- Plain language. "You can now..." not "Refactored the..."
- Never mention TODOS.md, internal tracking, eval infrastructure.
- Every entry should make someone think "oh nice, I want to try that."

### Bisectable commits at ship time

Even if the branch has messy commits from development, `/ship` produces a clean bisectable structure for the final push:

1. Infrastructure changes (migrations, config, dependencies).
2. Models/services (business logic).
3. Controllers/views (integration).
4. VERSION / CHANGELOG / TODOS (always last).

This way, if a production regression traces to this release, you can bisect to a single logical layer.

### Idempotency

`/ship` can be run multiple times on the same branch safely:

- If VERSION already bumped, skip the bump but read the version.
- If PR exists, update the body instead of creating a new one.
- If already pushed, skip the push.

### Verification gate

This is the rule that catches the most embarrassing bugs: **if code changed after the last test run, re-run tests before push.** Not "I'm confident." Run them.

```
Tests ran at step 3. Three more commits happened after step 6.
Re-running tests before push.
```

"Should work" is not evidence.

### Greptile triage

If the repo uses Greptile, its comments go through the same classification as review findings:

- **Valid** — added to critical findings, fixed.
- **Already fixed** — auto-reply acknowledging the catch.
- **False positive** — user confirms, reply explaining.
- **Suppressed** — silently skipped.

Patterns learn over time: confirmed false positives get saved and auto-skipped in future runs.

### Eval suites (conditional)

Only run if prompt-related files changed. Paid evals are expensive; don't burn budget on a CSS tweak. When they do run, they use the full judge tier — no Haiku shortcut on evaluations that gate a release.

---

## `/land-and-deploy` — PR to production

**Specialist:** Release engineer for the actual deploy.
**Job:** Merge the PR, wait for CI, wait for deploy, verify production health. One command from "approved" to "verified in production."

### What it does

```
1. Merge the PR (squash/rebase per repo convention)
2. Wait for CI to pass on main
3. Trigger deploy (or wait for auto-deploy)
4. Wait for deploy to complete
5. Run health check against the production URL
6. Run canary check across key pages
7. Report: healthy or specific failure
```

### First-run dry run

First run on a new project triggers a dry-run walk-through — you verify the pipeline before it does anything irreversible. After that, it trusts the config and runs straight through.

### Setup (`/setup-deploy`)

Run once. Detects the platform (Fly.io, Render, Vercel, Netlify, Heroku, GitHub Actions, or custom), discovers the production URL and health check endpoints, writes config to `CLAUDE.md`. One-time, 60 seconds.

### Example

```
Merging PR #42...
CI: 3/3 checks passed
Deploy: Fly.io — deploying v2.1.0...
Health check: https://myapp.fly.dev/health → 200 OK
Canary: 5 pages checked, 0 console errors, p95 < 800ms

Production verified. v2.1.0 is live.
```

### When deploys fail

If the deploy breaks, `/land-and-deploy` tells you what failed and whether to rollback. It doesn't auto-rollback — that's a user decision, because rollback is destructive.

---

## `/canary` — watch the live site

**Specialist:** SRE.
**Job:** After deploy, watch for trouble. Loop through key pages using the browse daemon. Check for console errors, performance regressions, page failures, visual anomalies. Take periodic screenshots.

Use it right after `/land-and-deploy`, or schedule it after a risky deploy.

### Example

```
Monitoring 8 pages every 2 minutes...

Cycle 1: ✓ All pages healthy. p95: 340ms. 0 console errors.
Cycle 2: ✓ All pages healthy. p95: 380ms. 0 console errors.
Cycle 3: ⚠ /dashboard — new console error:
         "TypeError: Cannot read property 'map' of undefined"
         at dashboard.js:142
         Screenshot saved.

Alert: 1 new console error after 3 monitoring cycles.
```

Canary is low-drama by design. It watches, alerts, screenshots. It doesn't auto-rollback. Humans (or explicit human-approved automation) decide what to do with alerts.

---

## `/benchmark` — performance baseline

**Specialist:** Performance engineer.
**Job:** Baseline page load times, Core Web Vitals (LCP, CLS, INP), resource counts, total transfer size. Run before and after a PR to catch regressions.

Uses the browse daemon for real Chromium measurements, not synthetic estimates. Multiple runs averaged. Results persist so you can track trends across PRs.

### Example

```
Benchmarking 5 pages (3 runs each)...

/           load: 1.2s  LCP: 0.9s  CLS: 0.01  resources: 24 (890KB)
/dashboard  load: 2.1s  LCP: 1.8s  CLS: 0.03  resources: 31 (1.4MB)
/settings   load: 0.8s  LCP: 0.6s  CLS: 0.00  resources: 18 (420KB)

Baseline saved. Run again after changes to compare.
```

The key is **persist across PRs**. A performance regression caught mid-PR is cheap to fix. One caught two weeks later — after 20 PRs have landed — is a forensic investigation.

---

## `/document-release` — keep docs in sync

**Specialist:** Technical writer.
**Job:** After `/ship` creates the PR but before it merges, read every documentation file and cross-reference against the diff. Update file paths, command lists, project structure trees, anything else that drifted.

This is the most underrated skill in the whole toolkit. Docs that lie about the product are worse than no docs. AI-generated code ages faster than human code; docs about it age at the same rate.

### What it updates

- `README.md` — skill counts, command lists, quickstart.
- `CLAUDE.md` — project structure trees, available skills, guidelines.
- `CONTRIBUTING.md` — dev setup, testing instructions.
- `TODOS.md` — marks completed items, adds new ones.
- `CHANGELOG.md` — polishes voice (without ever overwriting entries).
- Any `*.md` file that references paths the diff changed.

### Example

```
Analyzing 21 files changed across 3 commits. Found 8 documentation files.

README.md: updated skill count from 9 to 10, added new skill to table
CLAUDE.md: added new directory to project structure
CONTRIBUTING.md: current — no changes needed
TODOS.md: marked 2 items complete, added 1 new item

All docs updated and committed. PR body updated with doc diff.
```

Risky or subjective changes get surfaced as questions — voice changes, deleted sections, major rewrites. Safe changes (path updates, count updates, adding new entries) are handled automatically.

### `/ship` auto-invokes it

`/ship` automatically runs `/document-release` after creating the PR, so docs stay current without an extra command.

---

## E2E evals: don't claim "unrelated" without receipts

When an E2E eval fails during `/ship`, **never claim "not related to our changes" without proof.** These systems have invisible couplings: a preamble change affects agent behavior, a new helper changes timing, a regenerated SKILL.md shifts prompt context.

Required before attributing a failure to "pre-existing":

1. Run the same eval on main (or base branch) and show it fails there too.
2. If it passes on main but fails on the branch — **it IS your change**. Trace the blame.
3. If you can't run on main, say "unverified — may or may not be related" and flag it as a risk in the PR body.

"Pre-existing" without receipts is a lazy claim. Prove it or don't say it.

---

## Long-running tasks: don't give up

When running evals, E2E tests, or any long-running background task, **poll until completion**.

The anti-pattern: "I'll be notified when it completes" → stop checking. The task finishes 30 minutes later, the session has moved on, and the result gets lost.

The correct pattern: poll every 3 minutes. Report progress at each check (which tests passed, which are running, any failures so far). The user wants to see the run complete, not a promise that you'll check later.

Full E2E suites can take 30-45 minutes. That's 10-15 polling cycles. Do all of them.

---

## Why the final mile deserves this much discipline

Shipping is where the leverage happens. A feature that's 95% built and never shipped is worth zero. A feature that ships on time, with correct docs, with a verified deploy, with a performance baseline, with a canary watching — that's a feature your users actually get.

Most of the wins in AI-era engineering come from **never skipping the boring part**. Human teams skipped it because it was tedious. AI doesn't get tired. The process runs every time.

---

## What to take from this chapter

1. **Shipping is five sub-phases.** Ship → land-and-deploy → canary → benchmark → document-release. Each produces the artifact the next reads.
2. **Test bootstrap is a one-time cost.** Worth it on day one.
3. **Coverage audit belongs in the PR body.** `Tests: 42 → 47 (+5 new)` is the most useful line in any PR.
4. **Bisectable commits at ship time.** Infrastructure → models → controllers → VERSION. Messy dev commits are fine; the final push is clean.
5. **CHANGELOG is branch-scoped and for users.** Don't touch old entries. Don't mention internal tracking. Every entry is a "here's what you can now do."
6. **Verification gate.** If code changed after tests ran, re-run before push. "Should work" is not evidence.
7. **`/document-release` closes the loop.** Docs that drift are worse than no docs. Run it on every ship.
8. **E2E failures aren't "pre-existing" without receipts.** Prove it on main first.
9. **Poll until complete.** Long-running tasks don't finish themselves.

The next chapter is what happens after shipping — the reflection and memory that compound across sprints.

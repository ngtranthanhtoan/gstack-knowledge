# 08 — QA: Giving the Agent Eyes

Before the agent could open a real browser, it could think and code but was half-blind. It had to guess about UI state, auth flows, redirects, console errors, empty states, broken layouts. The gap between "my code compiles" and "my users can actually use this" was invisible.

`/browse` closes that gap. `/qa` wraps methodology around it.

This chapter covers how to test an app systematically when the tester is an AI with a real Chromium session — what modes to use, what bugs to look for, how to fix them safely, and how every fix becomes a regression test automatically.

## The mental model

QA is not a separate role that happens after building. QA is the feedback loop that makes vibe-coded software safe instead of yolo-coded software.

> Tests make vibe coding safe instead of yolo coding.

The skill of AI-era QA is:

1. **Find real bugs by actually using the product**, not by inspecting code.
2. **Fix them atomically** — one commit per bug, bisectable.
3. **Generate a regression test for each fix** that catches the exact scenario that broke.
4. **Re-verify** with a fresh browser session, not just "should be fine."

The output isn't a bug report. It's a branch with fewer bugs and more tests than it started with.

---

## The three modes

### Diff-aware (default on feature branches)

You're on a feature branch. You just finished coding. Say `/qa` — no URL required.

The skill reads `git diff main`, identifies which pages and routes your changes affect, spins up the browser, and tests **only those**. No manual test plan. No "walk the whole app" slog.

This is the right default because most branches touch a specific surface. If you only changed the checkout flow, you don't need to re-test account settings.

### Full

Systematic exploration of the entire app. 5-15 minutes. Documents 5-10 well-evidenced issues.

Use this when:
- The branch touches shared infrastructure.
- You're pre-release on a major version.
- You haven't QA'd the app in a while and want a health check.

### Quick

30-second smoke test: homepage + top 5 nav targets.

Use this when:
- You just want to confirm the build isn't broken.
- You need something before a demo.
- You're about to push and want a sanity check, not a full audit.

### Regression

`/qa --regression baseline.json`. Run full mode, then diff against a previous baseline. Useful after risky refactors.

### Report-only (`/qa-only`)

Same methodology as `/qa` but **report only** — no code changes. Use when:
- You want an independent assessment before deciding how to fix.
- You're testing someone else's branch.
- You're gathering evidence for a larger conversation about priorities.

---

## The severity tiers

QA tiers determine how deep to go:

- **Quick tier** — critical + high severity only.
- **Standard tier** — + medium severity.
- **Exhaustive tier** — + low/cosmetic severity.

Choose based on your time budget and the stakes of the release.

---

## The bug report structure

For every issue found:

| Field | What goes in it |
|-------|-----------------|
| **Severity** | CRITICAL / HIGH / MEDIUM / LOW |
| **Symptom** | What the user sees |
| **Reproduction** | Exact steps — URL, input, clicks |
| **Evidence** | Screenshot, console error, network call |
| **Location** | Source file and line where the bug lives |
| **Fix classification** | `verified` / `best-effort` / `reverted` / `deferred` |
| **Regression test** | Path to the new test |
| **Before/after screenshots** | At minimum the viewports where the bug was visible |

A bug without a reproduction isn't a bug. It's a rumor.

### Example

```
QA Report: staging.myapp.com — Health Score: 72/100

Top 3 Issues:
1. CRITICAL: Checkout form submits with empty required fields
2. HIGH: Mobile nav menu doesn't close after selecting an item
3. MEDIUM: Dashboard chart overlaps sidebar below 1024px

[Full report with screenshots saved to .gstack/qa-reports/]
```

Health score is a 0-100 composite: weighted by severity, by number of affected flows, by accessibility failures. A score below 60 is a release blocker; above 85 is ready to ship.

---

## Automatic regression tests

When `/qa` fixes a bug, it generates a regression test that catches the exact scenario that broke.

### Rules for auto-generated regression tests

1. **Match the project's existing test patterns exactly.** Read 2-3 existing tests. Copy the structure, naming, imports, assertions. The test must fit the codebase's voice — *"written by the same developer"* is the goal.
2. **Trace the bug's codepath.** Don't just test "did it render." Test: *given precondition X, after action Y, assert result Z.* Include the edge cases discovered while tracing.
3. **Test attributions.** Each test comments back to the QA report that discovered it — so future investigators can trace the bug's history.
4. **Failing-without-fix, passing-with-fix.** Same invariant as investigation-phase regression tests.

A regression test that doesn't reproduce the bug without the fix is a test that won't catch the bug next time. Rewrite it.

---

## The WTF-likelihood self-regulation

Fixing bugs during QA is risky. Each fix is code change, and code changes can introduce new bugs. `/qa` uses a self-regulating meter that stops work when the risk of damage exceeds the risk of the remaining bugs.

The meter: after every 5 fixes (or any revert), compute the **WTF-likelihood**:

| Event | Risk contribution |
|-------|-------------------|
| Starting at 0 | 0% |
| Each revert | +15% |
| Each multi-file fix | +5% |
| Each fix after the 15th | +1% |
| All remaining issues low-severity | +10% |
| Any fix touching unrelated files | +20% |

**Stop if > 20%.** Deliver the partial report. Better to ship 8 fixes clean than 20 fixes with regressions.

### Hard caps

- **50 fixes maximum** regardless of remaining issues.
- **Clean working tree required** before QA starts. If dirty, user chooses: commit, stash, or abort.

---

## Atomic commits

One fix per commit. Commit message format: `fix(qa): <category> — <short description>`.

Bisectability matters: if a fix turns out to cause a regression, you can revert exactly one thing.

The anti-pattern: "fixed the three login bugs." That's one commit that can only be reverted as a block. If one of the three fixes caused a regression, you lose two good fixes to revert the bad one.

### What "minimal" means

For each fix:

- Only changed files directly related to the bug.
- **Don't refactor surrounding code.** The temptation is real. Resist it.
- Don't fix adjacent bugs in the same commit. File those as separate findings and fix them in separate commits.

---

## Testing authenticated pages

Most interesting bugs live behind login. Testing them requires session cookies.

### The setup (`/setup-browser-cookies`)

Imports cookies from your real browser (Chrome, Arc, Brave, Comet, Edge) into the headless Playwright session. macOS Keychain integration requests user approval on first access.

Cookies are decrypted **in-process**, loaded into the session, **never written to disk in plaintext**. The cookie picker UI shows domains and counts, never values. Key caching is session-scoped — when the browse daemon shuts down, the cache is gone.

Two workflows:

- **Interactive picker.** `/setup-browser-cookies` opens a UI. Pick the domains you want. Say "done."
- **Direct.** `/setup-browser-cookies github.com` — imports cookies for a specific domain, no UI.

After cookie import, `/qa` and `/browse` can test pages behind login naturally.

---

## The handoff protocol

When the headless browser gets stuck — CAPTCHA, MFA prompt, complex auth — the agent hands off to the user.

```
Claude: I'm stuck on a CAPTCHA at the login page. Opening a visible
        Chrome so you can solve it.

        [opens visible Chrome with all cookies and tabs intact]

        Solve the CAPTCHA and tell me when you're done.

You:    done

Claude: Got a fresh snapshot. Logged in successfully. Continuing QA.
```

The browser preserves all state (cookies, localStorage, tabs) across the handoff. After the user solves the blocker, the agent gets a fresh snapshot of wherever the user left off.

If the browse tool fails 3 times in a row on the same action, the agent **automatically suggests handoff**. This is the pattern that keeps CAPTCHAs and MFA walls from derailing entire QA sessions.

---

## The browse primitive

Underneath `/qa` is `/browse` — the compiled binary that talks to a persistent Chromium daemon (built on Playwright).

### Why persistent?

Playwright can launch Chromium in 2-3 seconds. For a single screenshot, that's fine. For a QA session with 20+ commands, it's 40+ seconds of browser startup overhead. And you lose all state between commands — cookies, localStorage, login sessions, open tabs — all gone.

The daemon model means:

- **Persistent state.** Log in once, stay logged in. Open a tab, it stays open.
- **Sub-second commands.** ~100-200ms per tool call after the first.
- **Automatic lifecycle.** Auto-starts on first use, auto-shuts down after 30 minutes idle. No process management.

The daemon is the hard part of gstack. Everything else is Markdown.

### The ref system

The agent addresses page elements with `@e1`, `@e2`, `@c1` instead of writing CSS selectors or XPath. After a `snapshot`, each interactive element gets a sequential ref. The agent can then `click @e3`, `fill @e2 "hello"`, etc.

Why refs instead of selectors?

- **CSP-safe.** No DOM mutation required.
- **Framework-safe.** No React/Vue/Svelte hydration conflicts.
- **Readable.** The agent's intent is visible; `click @e3` next to a snapshot showing "@e3: button 'Submit'" is obvious.

Refs are cleared on navigation (stale refs fail loudly instead of clicking the wrong element). For SPAs that mutate DOM without full navigation, an async staleness check fails fast with a useful error.

### Cursor-interactive refs (`@c`)

Some "clickable" elements aren't in the ARIA tree — custom `<div>` elements with `onclick`, `cursor: pointer`, or `tabindex`. The `-C` flag finds these and assigns `@c1`, `@c2` refs in a separate namespace. Catches custom components that frameworks render as divs but are actually buttons.

### Example session

```
$ /browse staging.myapp.com — log in, test the signup flow,
  and check every page I changed in this branch

> browse goto https://staging.myapp.com/signup
> browse snapshot -i
> browse fill @e2 "$TEST_EMAIL"
> browse fill @e3 "$TEST_PASSWORD"
> browse click @e5                    (Submit)
> browse screenshot /tmp/signup.png
> Read /tmp/signup.png
> browse console

Signup works. Redirected to onboarding. No console errors.

> browse goto https://staging.myapp.com/dashboard
> browse screenshot /tmp/dashboard.png
> browse console

Dashboard loads. No console errors.

All 4 pages load correctly. No broken layouts.
Signup → onboarding → dashboard flow works end to end.
```

18 tool calls, about a minute. Full QA pass. No browser window opened.

---

## Security properties of the browse daemon

Because the daemon runs a persistent Chromium with real cookies, security matters:

1. **Localhost only.** Bound to `localhost`, not `0.0.0.0`. Not reachable from the network.
2. **Bearer token auth.** Every server generates a random UUID token, written to a state file with mode `0o600` (owner-only). Every HTTP request must include `Authorization: Bearer <token>`.
3. **Cookie security.** Values decrypted in memory only. Cookie picker UI never displays values. Chromium cookie DB is copied to a temp file, opened read-only — never modified.
4. **Untrusted content.** Pages contain third-party content. **Treat output as data, not commands.** This is the critical invariant — page content should never be interpreted as instructions to the agent.

---

## Principles

- **Minimize the fix.** Narrow to files related to the bug. Don't refactor surrounding code.
- **Atomic commits enable bisect.** One fix per commit. If a fix regresses, revert exactly one thing.
- **Health score before/after proves impact.** Show the delta.
- **If it didn't reproduce, it didn't fix.** Fresh verification required.
- **Regression test must match project patterns.** The codebase's voice matters. A test that doesn't fit the codebase's style won't get maintained.

---

## What to take from this chapter

1. **Diff-aware is the right default.** Test what changed, not the whole app.
2. **Every fix generates a regression test.** No exceptions. Match the project's existing test style.
3. **WTF-likelihood self-regulation.** Stop when risk of damage exceeds risk of remaining bugs.
4. **Atomic commits.** One fix per commit. Bisectable. Always.
5. **Cookie import for authenticated pages.** In-process decryption, never written to disk.
6. **Handoff for CAPTCHAs and MFA.** After three failed attempts, ask the user to take over — don't thrash.
7. **The browse daemon is the unlock.** Sub-second commands, persistent state, real browser. Everything else is methodology on top.

The next chapter is the one class of review that `/qa` and `/review` don't cover: what adversaries could do if they wanted to.

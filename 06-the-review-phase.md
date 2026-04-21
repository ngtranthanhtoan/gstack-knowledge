# 06 — Review: The Paranoid Staff Engineer

> Passing tests do not mean the branch is safe.

Code review exists because there's a whole class of bugs that survive CI and still punch you in the face in production. This chapter is about the review that asks: **what can still break?**

It's a structural audit, not a style nitpick pass. It's not about dreaming bigger or making the plan prettier. It's about imagining the production incident before it happens.

## Fix-first, not read-only

The old model of review: leave a list of comments. Developer eventually gets around to them. Half slip through.

The new model: **findings get action, not just listed.** Obvious mechanical fixes (dead code, stale comments, N+1 queries, missing null checks in trivial cases) get applied **automatically**. You see:

```
[AUTO-FIXED] app/services/user.ts:47 — Missing null check on email → added `?.` optional chain
[AUTO-FIXED] app/services/user.ts:89 — Dead branch after early return → removed 4 unused lines
```

Genuinely ambiguous issues (security, race conditions, design decisions) get surfaced for your call, one `AskUserQuestion` at a time — not batched into a wall of findings you have to triage.

The default is action. The exceptions are:

- Anything security-related: always ASK.
- Anything touching a test stub field: always ASK.
- Anything that changes observable behavior: always ASK.

Everything else: AUTO-FIX and show the diff.

## The categories of bugs to hunt

This is the working checklist. Every review runs through all of them; you can have an agent fan out in parallel or run them sequentially. The point is that the *categories* are domain knowledge — they're the bug classes that cost companies the most.

### SQL & data safety

- **N+1 queries.** The classic. Rendering a list that fires one query per row.
- **Stale reads.** Reading before a related write commits.
- **Missing indexes.** New WHERE clause on a column with no index.
- **Parameterized queries.** Every `$` or `?` is safe. Every string concatenation is a bug.
- **Race conditions on UPDATEs.** Is there a WHERE clause constraining to an expected prior state? Optimistic lock?
- **Orphaned records.** Failed jobs leaving storage in a dangling state.

### Trust boundaries

- **LLM output going into DB writes without validation.** Type-check against schema. The LLM can produce anything.
- **Client-provided metadata.** File type, user ID, permissions — if it came from the client, validate it server-side.
- **User input in prompts.** Prompt injection is a data-flow problem: does untrusted input reach a position that treats it as instructions?
- **Trust inheritance.** A function called by authed code may be called by unauthed code later. Don't assume inherited trust.

### Concurrency

- **Race conditions.** Two tabs, two requests, two jobs — can they step on each other?
- **Advisory locks.** Payment handlers, exactly-one-hero-image rules, quota decrements — all need explicit locks, not implicit assumptions.
- **Retry logic.** Idempotency keys? Exactly-once semantics, or at-least-once with dedup?
- **Background jobs.** Duplicate prevention? Dead-letter handling?

### Completeness

- **Forgotten enum handlers.** Add a new status or type constant, and the review traces it through every switch statement and allowlist in the codebase, not just the files you changed. This is the **single most labor-intensive category** because enum completeness requires reading files outside the diff.
- **Null/empty/boundary handling.** For every new field or parameter: what if it's nil? Empty? Maximum length?
- **Error path coverage.** Every exception has a named rescue. Every rescue has a user-visible result.
- **Shortcut implementations where the complete version costs less than 30 minutes of AI time.** Called out explicitly as a *completeness gap*.

### Shell & external command safety

- Shell injection. Parameterized or user-provided strings passed directly?
- Environment variable tampering.
- File path traversal.

### Tests that lie

- Tests that pass while missing the real failure mode. The canonical case: a test that calls a function and asserts it returns something, without checking that it returned the *right* thing.
- Tests that depend on order.
- Tests that mock the code under test.

## The enum completeness pattern

This deserves its own section because it's the trickiest category.

When a new status, type, or role constant is added, the review must:

1. Identify every file that references *any sibling* of the new constant (the whole enum family).
2. Read each one to check if the new value is handled.
3. Surface files where it isn't.

Example: you add `SUBSCRIPTION_STATUS.PAUSED` to an enum that already has `ACTIVE`, `CANCELLED`, `TRIAL`. The review finds every switch statement, if-chain, allowlist, and UI rendering that enumerates the other three — and checks each one.

Most of the time, 80% handle the new case correctly through a fallthrough. The other 20% silently do the wrong thing. Those are the bugs.

## Confidence calibration

Before recommending a fix pattern, **verify it's current best practice**. Not outdated from a 2019 blog post. Not deprecated. Not a workaround that's been obsoleted by a built-in.

The search protocol from chapter 1 applies here too:

1. Search `"{framework} {pattern} {current year}"`.
2. Check official docs.
3. Only recommend a pattern if Layer 1 (tried-and-true) or Layer 3 (first-principles) confirms it.

And when citing a claim in the codebase — *"this is already handled"* — **cite the specific line or file doing the handling**. Never say "likely handled" or "probably tested." Either you can point at it or you can't.

## Completeness gaps

This is the review's bite on the Boil-the-Lake principle.

If a shortcut implementation is shipped and the complete version is a lake, not an ocean, **call it out**. Not as a nitpick — as a finding. Specifically:

- Approach A: 150 lines, handles all edge cases, 30 minutes of AI time.
- Approach B (shipped): 80 lines, handles happy path, 10 minutes of AI time.

The review flags this with the cost delta. The user decides — but now with the information in front of them that A costs 20 more minutes, not 20 more days.

## The fix-first workflow

```
1. Find all issues (parallel scan by category)
2. Classify each:
   - AUTO-FIX: mechanical, no behavior change
   - ASK: judgment call, security, or visible behavior
   - SKIP: false positive or intentional
3. Apply AUTO-FIX issues immediately. Show diff.
4. Batch ASK issues into single AskUserQuestion events (one per finding).
5. After each user approval, apply the fix, re-verify.
6. Commit bisectably:
   - One logical change per commit
   - Commit message: "fix(review): <category> — <specific-issue>"
7. Report final state with counts per category.
```

No commits, no push. The review's job is to produce bisectable commits, not to merge them.

## Why slop-scan is advisory, not blocking

gstack uses [slop-scan](https://github.com/benvinegar/slop-scan) to catch AI code quality patterns. Things like empty catches around file ops, redundant `return await`, or swallowed exceptions that hide real errors.

**But it's advisory, not a blocker.**

### What to fix (genuine quality)

- **Empty catches around file ops.** Use `safeUnlink()` (ignores ENOENT, rethrows EPERM/EIO). A swallowed EPERM in cleanup means silent data loss.
- **Empty catches around process kills.** Use `safeKill()` (ignores ESRCH, rethrows EPERM). A swallowed EPERM means you think you killed something you didn't.
- **Redundant `return await`.** Remove when there's no enclosing `try` block. Saves a microtask, signals intent.
- **Typed exception catches.** `catch (err) { if (!(err instanceof TypeError)) throw err }` is genuinely better than `catch {}` when the try block does URL parsing or DOM work.

### What NOT to fix (gaming, not quality)

- **String-matching on error messages.** `err.message.includes('closed')` is brittle. Playwright/Chrome can change wording anytime. If a fire-and-forget operation can fail for ANY reason and you don't care, `catch {}` is the correct pattern.
- **Adding comments to exempt pass-through wrappers.** Noise, not documentation.
- **Converting extension catch-and-log to selective rethrow.** Chrome extensions crash entirely on uncaught errors. If the catch logs and continues, that IS the right pattern.
- **Tightening best-effort cleanup paths.** Shutdown, emergency cleanup, and disconnect code should swallow all errors. A cleanup path that throws on EPERM means the rest of cleanup doesn't run — worse.

The rule: **We are AI-coded and proud of it. The goal is code quality, not passing as human.** Don't chase the score. Fix patterns that represent actual problems. Accept findings where the "sloppy" pattern is the correct engineering choice.

## Multi-model second opinion

When `/review` (Claude) finishes, consider running `/codex` (OpenAI's Codex CLI) on the same diff. Different training, different blind spots, different strengths.

Three modes:

### Review mode
Codex reads every changed file, classifies findings by severity (P1 critical, P2 high, P3 medium), returns a PASS/FAIL verdict. Any P1 = FAIL. Fully independent — Codex doesn't see Claude's review.

### Challenge mode
Adversarial. Codex actively tries to break the code. Edge cases, race conditions, security holes, assumptions that fail under load. Maximum reasoning effort. A penetration test for your logic.

### Consult mode
Open conversation with session continuity. Ask Codex anything about the codebase. Follow-up questions reuse the same session so context carries.

### Cross-model analysis

When both `/review` and `/codex` have reviewed the same branch, you get:

- **Overlap findings** — both caught it. High confidence it's real.
- **Unique to Codex** — different perspective than Claude's.
- **Unique to Claude** — different perspective than Codex's.

This is the "two doctors, same patient" approach. The unique findings from each are where bugs neither would catch alone live.

### Example

```
CODEX REVIEW: PASS (3 findings)
[P2] Race condition in payment handler — concurrent charges
     can double-debit without advisory lock
[P3] Missing null check on user.email before downcase
[P3] Token comparison not using constant-time compare

Cross-model analysis (vs /review):
OVERLAP: Race condition in payment handler (both caught it)
UNIQUE TO CODEX: Token comparison timing attack
UNIQUE TO CLAUDE: N+1 query in listing photos
```

Three findings overlap across models = highest confidence.  
Unique to Codex = Claude had a blind spot.  
Unique to Claude = Codex had a blind spot.

## Greptile integration (external async reviewer)

If the repo uses [Greptile](https://greptile.com) — a PR review service — gstack pipes its findings through the same triage:

- **Valid issues** — added to critical findings, fixed before shipping.
- **Already-fixed issues** — auto-reply acknowledging the catch.
- **False positives** — pushed back. User confirms. Reply goes out explaining why.

Every false positive a user confirms gets saved. Future runs auto-skip known FP patterns for that codebase. `/retro` tracks Greptile's batting average over time — signal-to-noise ratio improving?

## What to take from this chapter

1. **Review is a structural audit.** N+1, stale reads, race conditions, trust boundaries, missing indexes, broken invariants — not style.
2. **Fix-first, not read-only.** Mechanical issues auto-fix; judgment calls get one question each.
3. **Enum completeness is the hardest category.** New constants need cross-repo verification, not just in-diff.
4. **Confidence calibration is required.** Verify patterns are current. Cite the line that proves "handled."
5. **Slop-scan is advisory.** Fix what's a real quality issue. Accept findings where "sloppy" is correct.
6. **Multi-model second opinion catches blind spots.** Overlap = high confidence. Unique-to-each = where the bugs hide.
7. **No commits, no push.** The review produces bisectable commits. Merging is the ship phase.

Two chapters follow: one for when something is already broken and you need to debug it, one for when it all looks fine and you need to prove it.

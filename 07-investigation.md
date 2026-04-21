# 07 — Investigation: The Iron Law

Something is broken and you don't know why. This chapter is about what to do in that moment — and specifically, what *not* to do.

The default instinct for every developer, human or AI, is to propose a fix. See error, look at code, guess, patch, run, see new error, guess, patch. Within thirty minutes you've made a dozen small changes, the bug is still there, and you've lost track of what you actually tried.

The Iron Law breaks that cycle.

## The Iron Law

**No fixes without root cause investigation first.**

Fixing symptoms creates whack-a-mole. Every symptom-only fix makes the next bug harder to find because you're now looking at the output of a band-aid, not the original behavior.

If you take only one thing from this chapter: **when something breaks, understand it before you touch it.**

The remainder is a systematic methodology that enforces the Iron Law.

## The five phases

Investigation proceeds in five phases. Each phase produces an output the next phase reads. Skipping phases produces fast wrong answers.

```
1. Root Cause Investigation  → a testable hypothesis
2. Pattern Analysis          → match against known bug shapes
3. Hypothesis Testing        → confirm or disprove, maximum 3 tries
4. Implementation            → fix the cause, not the symptom
5. Verification & Report     → prove it's fixed, record what happened
```

---

## Phase 1 — Root Cause Investigation

**Output:** *"Root cause hypothesis: ..."* — a specific, testable claim.

Six steps:

### 1. Collect symptoms
Errors. Stack traces. Exact reproduction steps. What did the user expect? What happened instead? Screenshot or log if possible.

### 2. Read the code
Trace the path from symptom back toward causes. Don't skim — read. The bug is almost always in a place the stack trace doesn't directly point to.

### 3. Check recent changes
```
git log --oneline -20 -- <affected-files>
```
Was this working before? What changed? **A regression means the root cause is in the diff.** This is the single most productive step in most investigations.

### 4. Reproduce
Can you trigger it deterministically? If not, that's a hypothesis itself — maybe it's timing-dependent, environment-dependent, or data-dependent.

### 5. Check history
Any prior investigations on these files? Recurring bugs in the same area? **Recurring bugs = architectural smell, not coincidence.** If the module has three bugs in the last month, the module is wrong, not the code.

### 6. Form the hypothesis
One sentence. Specific. Testable. Starting with *"Root cause hypothesis: ..."*.

Bad: *"Something's wrong with the auth flow."*
Good: *"Root cause hypothesis: the session middleware runs before the CSRF middleware, so the CSRF token is read from an empty session on the first request."*

---

## Phase 2 — Pattern Analysis

Match the hypothesis against known bug patterns. Most bugs are instances of a small number of shapes.

| Pattern | Signature | Where to look |
|---------|-----------|---------------|
| **Race condition** | Intermittent, timing-dependent, "works on my machine" | Concurrent access to shared state |
| **Nil/null propagation** | `NoMethodError`, `TypeError`, "undefined is not a function" | Missing guards on optional values |
| **State corruption** | Inconsistent data, partial updates | Transactions, callbacks, hooks |
| **Integration failure** | Timeout, unexpected response | External API calls, service boundaries |
| **Configuration drift** | Works locally, fails in prod | Env vars, feature flags, DB state |
| **Stale cache** | Shows old data, fixes on cache clear | Redis, CDN, browser cache, Turbo |
| **Enum gap** | Works for known values, fails silently on new ones | Switch statements, allowlists |
| **Order-of-operation** | Effect happens before cause | Middleware order, callback order, async timing |

Also check: `TODOS.md` for related known issues. Prior fixes in the same area = architectural smell.

---

## Phase 3 — Hypothesis Testing

**Output:** Confirmed root cause, or a new hypothesis.

### 1. Confirm the hypothesis

Add a temporary log or assertion at the suspected root cause. Run the reproduction. Does the evidence match the hypothesis?

Two possible outcomes:

- **Evidence matches.** Root cause confirmed. Proceed to Phase 4.
- **Evidence doesn't match.** The hypothesis is wrong. Don't patch anyway — loop back to Phase 1 with what you learned.

### 2. If the hypothesis was wrong

Before forming the next hypothesis, search. But **sanitize first** — strip IPs, paths, SQL, PII. Search only the error type plus framework context. (A raw error message with PII is a leak.)

### 3. The 3-strike rule

**If three hypotheses fail, STOP.** Don't form a fourth.

This is where most debugging sessions spiral. The instinct is "one more try." Three failed hypotheses is strong evidence that the bug isn't a simple mismatch. It's architectural, or the mental model is wrong, or the symptom is misleading you.

When 3-strike triggers:

- Stop touching code.
- Use `AskUserQuestion` to escalate.
- Re-examine the architecture, not the implementation.

The 3-strike rule sounds arbitrary. It isn't. Empirically, if three specific hypotheses were all wrong, the bug lives at a layer above where you were looking.

---

## Phase 4 — Implementation

**Only enter this phase after Phase 3 confirms a root cause.**

### 1. Fix the root cause, not the symptom
The smallest change that eliminates the actual problem. Not a band-aid that masks it.

### 2. Minimal diff
Fewest files, fewest lines. **Don't refactor adjacent code.** Refactoring during investigation muddles the fix — you can't bisect later because your "bug fix" commit also changed three unrelated things.

### 3. Regression test
Write a test that:
- **Fails without the fix** (proves the test reproduces the bug).
- **Passes with the fix** (proves the fix works).

If the test passes without the fix, it's not testing the actual bug. Rewrite it.

### 4. Run the full test suite
No regressions allowed. The fix is wrong if it breaks something else.

### 5. Blast radius check
If the fix touches more than 5 files, flag it via `AskUserQuestion`. Large fixes during investigation usually mean the diagnosis was wrong or the fix is going beyond the bug.

---

## Phase 5 — Verification & Report

### Fresh verification
Reproduce the original bug. Confirm it's gone. Run tests. Paste the output.

Not *"should work now."* Run it.

### The structured debug report

```
Symptom:          [what the user observed]
Root cause:       [what was actually wrong]
Fix:              [what changed, file:line refs]
Evidence:         [test output proving it works]
Regression test:  [file:line of new test]
Related:          [TODOS, prior bugs in area]
Status:           DONE | DONE_WITH_CONCERNS | BLOCKED
```

The report isn't ceremony — it's how the **next** investigation of a related bug gets context for free. Three months from now, when a similar symptom shows up, this report is what someone (probably future you) reads.

---

## Data flow tracing

For subtle bugs where the symptom is far from the cause, trace data from entry point to failure.

1. Where does the data enter the system?
2. What transforms it?
3. Where does it get persisted or passed on?
4. Where does the symptom observe it?

Find the point where the value the code *has* diverges from the value the code *needs*. That point is the root cause.

This is especially useful for:

- Data corruption bugs.
- Wrong-value-displayed bugs.
- "Why is this field null?" bugs.

The naive approach — "look at the line where it's null" — is almost never the cause. The cause is upstream, often across a network or process boundary.

---

## Red flags during investigation

These are the phrases that indicate you're about to make the bug worse:

- **"Quick fix for now."** There is no "for now." Fix it right or escalate. "For now" fixes become permanent and hide the real issue.
- **"Let me try adding a try/catch around this."** Wrapping an error isn't a fix — it's a way to make the error less visible. If you don't know why it's throwing, a try/catch makes investigation harder.
- **"Proposing fix before tracing data flow."** You're guessing. Trace first.
- **"Each fix reveals a new problem elsewhere."** You're at the wrong layer. The real bug is one level up from where you're looking.
- **"I'll just retry it."** Retry is appropriate for known-flaky external calls. Retry is a band-aid for unknown timing bugs.

---

## The freeze pattern during investigation

When you're debugging a billing bug, you don't want the agent accidentally "fixing" unrelated code in `src/auth/`. `/investigate` automatically activates `/freeze` on the module under investigation — all edits are restricted to that directory.

This prevents the most common form of debug-session damage: the agent making drive-by "improvements" to files it was only supposed to read.

Chapter 12 covers `/freeze` and the related safety guardrails in detail.

---

## When the investigation fails

Sometimes the Iron Law produces a verdict you don't want: *"I can't reproduce deterministically, my three hypotheses were wrong, and the architecture looks clean."*

At that point, the right answer is:

1. Document what you tried, in the structured debug report.
2. Escalate to human eyes — or a second model via `/codex consult`.
3. Do not patch defensively. A defensive patch compounds the problem.

An acknowledged-unsolved bug is better than a hidden-still-exists bug. Hidden bugs resurface with worse symptoms and less context.

---

## What to take from this chapter

1. **The Iron Law: no fixes without root cause investigation.** Guessing and patching compounds bugs.
2. **Five phases.** Investigation → pattern → hypothesis → fix → verify. Each produces the input for the next.
3. **Check recent changes first.** Regression = root cause is in the diff.
4. **Recurring bugs in one area = architectural smell.** Not coincidence.
5. **3-strike rule.** If three hypotheses fail, stop and escalate. The bug is at a higher layer than you're looking.
6. **Fix the root cause, minimal diff.** Don't refactor adjacent code. Regression test required.
7. **Fresh verification, always.** "Should work" is not evidence. Run it.
8. **Structured debug report.** Future investigations read this. Document even the ones that didn't conclude.
9. **Recurring red flags.** "Quick fix." "Try/catch around it." "Just retry." These are how bugs get buried.

The next chapter is the complementary skill: what to do when no specific bug has been reported but you want to find them before users do.

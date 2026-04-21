# 12 — Safety and Parallel Sprints

The previous eleven chapters covered a single sprint — think, plan, build, review, test, ship, reflect. This closing chapter covers two orthogonal concerns that change the shape of the whole system: how to stay safe when the AI has hands, and how to scale the methodology to ten or fifteen concurrent sprints without drowning.

## Part 1 — Safety Guardrails

AI agents can now do what used to take a team of twenty. That includes doing real damage twenty times faster.

The guardrails are not access control. They are **accident prevention** — a set of tripwires that catch the cases where the agent is about to do something it shouldn't, and gives the user a chance to intervene.

Three skills cover the range: `/careful`, `/freeze`, `/guard`.

### `/careful` — destructive-command warnings

Say *"be careful"* or run `/careful` when you're working near production, running destructive commands, or just want a safety net. Every bash command gets checked against known-dangerous patterns.

#### The dangerous-command categories

| Pattern | Risk |
|---------|------|
| `rm -rf` / `rm -r` | Recursive delete |
| `DROP TABLE` / `DROP DATABASE` / `TRUNCATE` | Data loss |
| `git push --force` / `git push -f` | History rewrite |
| `git reset --hard` | Discard commits |
| `git checkout .` / `git restore .` | Discard uncommitted work |
| `kubectl delete` | Production resource deletion |
| `docker rm -f` / `docker system prune -a` | Container/image loss |

Before running any of these, the guardrail pauses and confirms.

#### Whitelisted exceptions (no warning)

Common build-artifact cleanups:

- `rm -rf node_modules`
- `rm -rf dist`
- `rm -rf .next`
- `rm -rf __pycache__`
- `rm -rf build`
- `rm -rf coverage`
- `rm -rf .turbo`, `.cache`

These are routine. Warning on every one trains the user to say "yes" reflexively — which then fails them when the warning actually matters.

#### Override semantics

Any warning can be overridden. The guardrails are **accident prevention, not access control**. The user is sovereign (chapter 1); the guardrail's job is to make sure they see what's about to happen before they approve.

### `/freeze` — directory-scoped edit lock

Restrict all file edits to a single directory. When you're debugging a billing bug, you don't want the agent accidentally "fixing" unrelated code in `src/auth/`.

```
/freeze src/billing
→ Edits restricted to src/billing/. Run /unfreeze to remove.

[later, agent tries to edit src/auth/middleware.ts]
→ BLOCKED — Edit outside freeze boundary (src/billing/). Skipping.
```

This blocks **Edit** and **Write** tool calls only. Bash commands like `sed` can still modify files outside the boundary — it's accident prevention, not a sandbox.

#### `/investigate` auto-freezes

Chapter 7 mentioned this: when you run `/investigate`, it automatically freezes to the module being debugged. This prevents the most common debug-session damage: the agent making drive-by "improvements" to files it was only supposed to read.

#### `/unfreeze`

Remove the boundary, allowing edits everywhere. The hooks stay registered for the session so `/freeze` can re-activate with a different boundary.

### `/guard` — both at once

Combines `/careful` + `/freeze` in one command. Destructive-command warnings **plus** directory-scoped edits. Use when touching production or debugging live systems.

The pattern: when you're about to do risky work, prefix with `/guard` and the boundary. Both guardrails activate, and you get both layers of protection.

### Principles

- **Warnings are not blockers.** Override them when you mean to.
- **Whitelists prevent alarm fatigue.** Common safe operations shouldn't trigger warnings.
- **Accident prevention, not access control.** Don't confuse the goal.
- **Investigate auto-freezes.** Scope creep during debugging is the most common damage vector.

---

## Part 2 — Parallel Sprints

> gstack is powerful with one sprint. It is transformative with ten running at once.

The sprint process in chapters 2-11 is designed for a single cognitive thread: you think, plan, build, review, test, ship, reflect. But AI makes parallel cognitive threads cheap. The practical maximum right now is 10-15 parallel sprints.

### Why parallelism works with process, not without

Without process, ten agents is **ten sources of chaos**. Each is making independent decisions, checking into the same branch or adjacent branches, stepping on each other's changes, making contradictory recommendations.

With process, each agent knows **exactly what to do and when to stop**. `/office-hours` stops at a design doc. `/plan-eng-review` stops at a locked plan. `/review` stops at findings-for-approval. Each phase has a clear exit.

You manage the agents the way a CEO manages a team: check in on the decisions that matter, let the rest run.

### The workspace-per-sprint model

Tools like [Conductor](https://conductor.build) run multiple Claude Code sessions in parallel — each in its own isolated workspace (a git worktree). One session runs `/office-hours` on a new idea, another does `/review` on a PR, a third implements a feature, a fourth runs `/qa` on staging, and six more on other branches.

The isolation matters:

- **No accidental cross-branch edits.** Each workspace is a worktree. Files don't leak.
- **Parallel test runs without interference.** Two `/ship` runs can happen simultaneously.
- **Mental model scales.** Each window is one sprint in one phase. You context-switch between windows, not within a window.

### What you watch vs. what you let run

The rule: **check in on the decisions, not the execution.**

**Worth watching:**
- `/office-hours` reframes — these shape the entire project.
- `/plan-ceo-review` scope decisions — expansion, hold, reduction.
- `/plan-eng-review` architecture tradeoffs — these compound for months.
- `/review` ASK items — security, race conditions, design decisions.
- `/qa` bug classification — critical vs high vs medium.
- Any `AskUserQuestion` prompt from any skill.

**Not worth watching:**
- `/build` implementations when the plan is locked.
- `/ship` runs when tests are green.
- `/canary` cycles on healthy deploys.
- Retry loops on transient failures.
- Most of the sub-agent dispatch (coverage audits, completion checks).

### Smart review routing

A good startup has an unwritten rule: the CEO doesn't look at infra bug fixes, design review isn't needed for backend changes, eng manager doesn't need to weigh in on copy edits. Each review happens only when it's the appropriate lens.

gstack encodes this routing. `/autoplan` runs only the reviews that apply based on what the plan touches:

- UI changes → design review activates.
- API/CLI/SDK changes → DX review activates.
- Backend logic → eng review, skip design.
- All of the above → full gauntlet.

The Review Readiness Dashboard tells you where you stand before you ship:

```
Eng Review:    CLEAR
CEO Review:    CLEAR (informational)
Design Review: —     (not applicable to this branch)
```

### ELI16 mode for overwhelmed humans

When 3+ sessions are running simultaneously, every skill enters **ELI16 mode** — every question re-grounds the user on context because they're juggling windows.

Instead of *"Approve the race condition fix?"* the question becomes:

> *"In project `billing-api`, branch `fix/payment-charges`, the review found a race condition in `payment_service.rb:47` where concurrent charges can double-debit. The fix adds an advisory lock. Approve?"*

The extra context is noise when you're on one window. It's essential when you're on three.

### Voice input as a force multiplier

When running 10 parallel sessions, the bottleneck is typing. Voice input (AquaVoice, Whisper, etc.) changes the math entirely.

gstack skills have **voice-friendly trigger phrases**. Say what you want naturally — *"run a security check," "test the website," "do an engineering review"* — and the right skill activates. No need to remember slash command names or acronyms.

### Proactive skill suggestions

The agent notices what stage you're in — brainstorming, reviewing, debugging, testing — and suggests the right skill. Don't like it? Say "stop suggesting" and it remembers across sessions.

### Cross-agent coordination (`/pair-agent`)

You're in Claude Code. You also have OpenClaw running. Or Hermes. Or Codex. You want them both looking at the same website.

`/pair-agent` opens a shared browser with session isolation:

1. Type `/pair-agent`, pick your agent.
2. A GStack Browser window opens so you can watch.
3. The skill prints a block of instructions. Paste that block into the other agent's chat.
4. It exchanges a one-time setup key for a session token, creates its own tab, and starts browsing.
5. You see both agents working in the same browser, each in their own tab. Neither can interfere with the other.

Security properties:
- **Scoped tokens.** Each agent gets its own token with limited capabilities.
- **Tab isolation.** An agent can only interact with its own tab.
- **Rate limiting.** Per-agent caps on command frequency.
- **Domain restrictions.** Agents can be constrained to specific domains.
- **Activity attribution.** Every command is logged with the agent that issued it.

If ngrok is installed, the tunnel starts automatically so the other agent can be on a completely different machine. Same-machine agents get a zero-friction shortcut.

This is the first time AI agents from different vendors can coordinate through a shared browser with real security.

### The sidebar agent (for personal automation)

`/open-gstack-browser` launches a rebranded Chromium with a sidebar extension. Type natural language in the side panel and a child Claude instance executes it:

- *"Navigate to the settings page and screenshot it."*
- *"Fill out this form with test data."*
- *"Go through every item in this list and extract the prices."*

Auto-routes to the right model: Sonnet for fast actions (click, navigate, screenshot) and Opus for reading and analysis. Each task gets up to 5 minutes.

The sidebar agent is isolated from the main Claude Code session, so it won't interfere with your main window. One-click cookie import from the sidebar footer.

Use cases beyond dev workflows: *"Browse my kid's school parent portal and add all the other parents' names, phone numbers, and photos to my Google Contacts."* Log in once in the headed browser, session persists. Or click the "cookies" button to import from your real Chrome. Once authenticated, the agent navigates the directory, extracts the data, and creates the contacts.

### Browser handoff for blockers

Hit a CAPTCHA, auth wall, or MFA prompt? The browser handoff protocol (chapter 8) works across parallel sessions too. After 3 consecutive failures, the agent automatically suggests handoff. Solve the blocker in a visible Chrome, say done, the agent resumes at the same page with all state intact.

---

## Part 3 — The architecture notes that make all this possible

A few specific technical decisions enable the parallelism and safety described above. They're worth knowing even if you don't implement them yourself.

### The daemon model

Every command you run against a browser is an HTTP POST to a long-lived Chromium daemon. First call starts the browser (~3s). Every call after: ~100-200ms.

Without the daemon, every command would cold-start a new browser. You'd lose cookies, tabs, and login state between commands. A 20-command QA session would cost 40 seconds of browser startup alone.

### Random port selection

The daemon binds to a random port between 10000-60000, retrying up to 5 times on collision. This means 10 Conductor workspaces can each run their own browse daemon with zero configuration and zero port conflicts.

The old approach (scanning 9400-9409) broke constantly in multi-workspace setups.

### Localhost + bearer token

The HTTP server binds to `localhost` (not `0.0.0.0`), and every request requires a bearer token written to a state file with mode `0o600`. This prevents other processes on the same machine from talking to your browse server.

Multi-user is explicitly not supported. One server per workspace, one user. The token is defense-in-depth, not multi-tenancy.

### Atomic writes to state files

The state file (`pid`, `port`, `token`, `startedAt`, `binaryVersion`) is written via tmp + rename, mode `0o600`. This means if the writer is interrupted, readers never see a partially-written file.

This pattern — atomic writes, atomic renames — shows up everywhere in parallel-safe systems. Use it whenever multiple processes might read the same file.

### Version auto-restart

Every build writes the git SHA to `.version`. On each CLI invocation, if the binary's version doesn't match the running server's, the CLI kills the old server and starts a new one.

This prevents the "stale binary" class of bugs entirely. Rebuild, next command picks it up. No manual process management.

### The ref system (chapter 8 recap)

Element references (`@e1`, `@e2`, `@c1`) address page elements by their position in the accessibility tree, not CSS selectors. Stale refs fail loudly on the next command (async count check). The agent knows to run `snapshot` again.

This is the difference between "element not found" and the agent silently clicking the wrong element because the selector matched something else.

### Error philosophy

Errors are for AI agents, not humans. Every error message is actionable:

- Native Playwright: *"Element not found."*
- Rewritten: *"Element not found or not interactable. Run `snapshot -i` to see available elements."*
- Native: *"Navigation timeout."*
- Rewritten: *"Navigation timed out after 30s. The page may be slow or the URL may be wrong."*

The agent should read the error and know what to do next without human intervention.

---

## The compound effect

Chapter 1 said the delta isn't that anyone became a better programmer — it's that AI let people actually ship the things they always wanted to build.

The compound effect across a year looks like this:

- 60-day cycle from idea to user, instead of 3 months.
- 10-15 parallel sprints, instead of 1-2 serial ones.
- Every sprint reads the last sprint's learnings, so decisions get faster.
- Every bug found gets a regression test, so the same bug doesn't recur.
- Every retro surfaces a pattern that goes into memory, so the next sprint avoids it.
- Every ship runs a canary, so a regression in production gets caught in minutes, not weeks.

None of this is exotic. It's just not-skipping-anything. Humans skipped the boring parts because they were boring. AI doesn't get bored. The process runs every time. The compound is what makes a single person shipping like a team of twenty possible.

---

## What to take from this chapter

1. **Safety guardrails are accident prevention, not access control.** Override when you mean to.
2. **Destructive-command warnings have a whitelist.** Common cleanups shouldn't trigger.
3. **Freeze locks edits to a directory.** `/investigate` auto-freezes to the module under debug.
4. **Parallelism needs process.** Ten agents without structure is ten sources of chaos.
5. **Each workspace is a worktree.** Isolation prevents cross-branch damage.
6. **Watch decisions, not execution.** Scope, architecture, security calls. Not build or ship cycles.
7. **Cross-agent coordination with scoped tokens.** Different vendors, same browser, tab isolation.
8. **The daemon model is the unlock.** Persistent state, sub-second commands, random ports for concurrent workspaces.
9. **Errors are for agents.** Actionable messages tell the agent what to do next.
10. **The compound effect is the story.** Not raw LOC. Not any one skill. The whole process not skipping any step.

---

## Closing

This book is one person's methodology written down. It was built because its author wanted it, shaped by how he works, and opened up because tools like this should be available to everyone. Fork it, improve it, ignore the parts that don't fit your context.

The ground moved. The process here is one attempt at keeping up with it without losing the things that matter: taste, rigor, correctness, and the willingness to do the complete thing.

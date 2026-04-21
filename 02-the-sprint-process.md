# 02 — The Sprint Process

> gstack is a process, not a collection of tools.

Think of a startup sprint as a sequence of distinct cognitive modes. Each mode has a job. Each mode produces an artifact the next mode reads. Skip a mode and the next one has to guess — usually badly.

The seven phases:

**Think → Plan → Build → Review → Test → Ship → Reflect**

Each skill in gstack corresponds to one phase. Each phase has a specialist voice. The reason the whole thing works is that the artifacts chain together — `/office-hours` writes a design doc that `/plan-ceo-review` reads; `/plan-eng-review` writes a test plan that `/qa` picks up; `/review` catches bugs that `/ship` verifies are fixed. **Nothing falls through the cracks because every step knows what came before it.**

## Why a process at all

With AI, a single person can now do what a team used to do. But a team had structure: a product manager thought about the user, an architect thought about the system, an eng lead thought about the code, QA thought about failure, a release manager thought about shipping. Those were separate *people* enforcing separate *modes of thought*.

Without process, AI just blurs all those roles into one over-eager assistant that builds, reviews, tests, and ships simultaneously — and none of them very well. The agent becomes ten sources of chaos instead of one focused collaborator.

gstack's insight: **give the agent structure by giving it explicit phase changes**. When you run `/office-hours`, it is *not* thinking about tests. When you run `/ship`, it is *not* dreaming bigger. Each mode is a different intelligence, and the handoffs between modes are the rails that keep AI from spiraling.

## The seven phases

### 1. Think — what are we actually building?

**Specialist:** YC Office Hours partner.
**Tools:** `/office-hours`.
**Input:** A rough idea, a feature request, a pain point.
**Output:** A design doc with reframed problem, validated premises, and 2-3 implementation approaches with effort estimates.

This is the phase most people skip and pay for later. You describe what you want to build, and an interrogator pushes back until you've named the real problem. "Daily briefing app" turns out to be "personal chief of staff AI." The reframe saves you from spending three weeks building the wrong thing.

Two sub-modes: **startup mode** (six forcing questions about demand, status quo, the specific human) and **builder mode** (enthusiastic collaboration on the coolest version of the idea). Both end with a design doc that downstream phases can read.

**Chapter 3** covers the methodology in detail.

### 2. Plan — make it buildable

**Specialists:** CEO / Founder, Eng Manager, Senior Designer, DevEx Lead.
**Tools:** `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review`, `/plan-devex-review`, or `/autoplan` to run all of them with encoded decision principles.
**Input:** The design doc from think-phase.
**Output:** A locked-down plan with architecture diagrams, test matrix, failure modes, interaction states, and distribution strategy.

Four different kinds of intelligence in sequence:

- **CEO review** rethinks the problem. What's the 10-star product hiding inside this request? Four scope modes: expansion, selective expansion, hold, reduction.
- **Eng review** locks architecture. Forces hidden assumptions into diagrams. Maps failure modes, edge cases, test coverage.
- **Design review** rates the plan on seven design dimensions 0-10. Catches "AI slop" language ("cards with icons", "hero with gradient") before a single pixel is committed.
- **DevEx review** asks how developers will meet this product. Time-to-hello-world, error message tiers, magical moment.

The point of doing all four is that they surface different failures. CEO catches "we're building the wrong thing." Eng catches "this will deadlock under load." Design catches "no empty state defined." DevEx catches "the getting-started path has seven steps."

**Chapter 4** covers each specialist review. **Chapter 5** covers the design pipeline end-to-end.

### 3. Build — write the code

**Specialist:** Just your AI assistant, now with an approved plan and an architecture diagram.
**Tools:** Claude Code, Cursor, Hermes, OpenClaw — any coding agent.
**Input:** The plan.
**Output:** A feature branch with code that matches the plan.

This phase has the least structure in gstack because the previous phases did the structural work. With a good plan, implementation is mostly mechanical. The agent knows what to build, where it fits, what tests to write, and what edge cases to cover.

Two principles matter here:

- **Confusion Protocol.** If the agent encounters an ambiguity the plan didn't resolve, it stops and asks rather than guessing. Most "AI slop" bugs come from guessing.
- **Test-first when possible.** The plan's test matrix becomes real tests. If the test was specified, write it before the code.

### 4. Review — what can still break?

**Specialist:** Paranoid staff engineer. Optionally, a second model (Codex) for cross-model review.
**Tools:** `/review`, `/codex`.
**Input:** A feature branch ready to ship.
**Output:** A classified list of findings, with mechanical issues auto-fixed and judgment calls surfaced for your approval.

> Passing tests do not mean the branch is safe.

This is a structural audit, not a style nitpick pass. The questions are deliberately different from the earlier phases: N+1 queries, stale reads, race conditions, bad trust boundaries, missing indexes, escaping bugs, broken invariants, bad retry logic, tests that pass while missing the real failure mode.

Fix-first, not read-only. Obvious mechanical fixes get applied automatically. Judgment calls get one `AskUserQuestion` per finding. Completeness gaps (the 80% solution when the 100% solution is a lake) get called out.

**Chapter 6** covers the methodology.

### 5. Test — give the agent eyes

**Specialist:** QA lead with a real browser.
**Tools:** `/qa`, `/qa-only`, `/browse`.
**Input:** A feature branch and optionally a staging URL.
**Output:** A bug report, atomic commits for each fix, a regression test for each bug, and before/after screenshots.

Before `/browse`, the agent could think and code but was half-blind. It had to guess about UI state, auth flows, redirects, console errors, empty states, broken layouts. Now it can just go look.

Three testing modes:

- **Diff-aware** (default on feature branches) — reads `git diff main`, identifies affected pages, tests them specifically.
- **Full** — systematic exploration. 5-15 minutes, 5-10 well-evidenced issues.
- **Quick** — 30-second smoke test: homepage + top 5 nav targets.

Every bug `/qa` fixes becomes a regression test. Every regression test is written in the codebase's existing test style. Traces back to the QA report for attribution.

**Chapter 8** covers the methodology.

### 6. Ship — finish the job

**Specialists:** Release engineer, SRE, technical writer.
**Tools:** `/ship`, `/land-and-deploy`, `/canary`, `/benchmark`, `/document-release`.
**Input:** A reviewed, tested branch.
**Output:** A merged PR, a verified-in-production deploy, updated docs, a canary report.

This is the phase where human-coded branches go to die. The interesting work is done. Only the boring release work is left. Humans procrastinate; AI should not.

`/ship` syncs main, runs tests, audits coverage, updates changelog/version, pushes, opens or updates the PR. If your project has no test framework, `/ship` bootstraps one and writes real tests for your actual code. If code changed after tests were last run, it re-runs them.

`/land-and-deploy` merges the PR, waits for CI, deploys, verifies health. One command from "approved" to "verified in production."

`/canary` watches the live site for console errors, performance regressions, and new failures. `/benchmark` baselines Core Web Vitals and tracks regressions PR over PR. `/document-release` reads every doc file and updates anything that drifted.

**Chapter 10** covers the methodology.

### 7. Reflect — what actually happened?

**Specialist:** Engineering manager.
**Tools:** `/retro`, `/learn`.
**Input:** The week's commits, PRs, tests, and ships.
**Output:** A data-grounded retrospective with metrics, streaks, and growth opportunities per contributor; updated institutional memory.

This phase is where AI-assisted work usually gets sloppy. Without reflection, patterns compound silently: test coverage slips, the same bug class recurs, one contributor's PRs keep getting rewritten.

`/retro` computes metrics (weighted commits, logical SLOC, test ratio, focus score, fix ratio), detects coding sessions by commit timestamps, tracks shipping streaks, identifies hotspot files. Team-aware: it gives you the deepest treatment on your own work and breaks down each contributor with specific praise and specific growth opportunities.

`/learn` is the memory layer. Patterns, pitfalls, preferences, architectural decisions accumulate in a project-specific journal. Each learning has a confidence score. Stale learnings get pruned when the files they reference no longer exist. Other skills automatically search learnings before making recommendations, and display "Prior learning applied" when a past insight matters.

**Chapter 11** covers retros and memory.

## The sprint at a glance

```
  ┌─────────────────────────────────────────────────────────┐
  │ THINK                                                    │
  │   /office-hours                                          │
  │   → writes design doc to ~/.gstack/projects/             │
  └────────────────────┬────────────────────────────────────┘
                       │
                       ▼
  ┌─────────────────────────────────────────────────────────┐
  │ PLAN                                                     │
  │   /plan-ceo-review   (scope)                             │
  │   /plan-design-review (interaction states, AI slop)      │
  │   /plan-eng-review   (architecture, tests, failure modes)│
  │   /plan-devex-review (if building for developers)        │
  │     — or /autoplan to run them all automatically         │
  │   → writes locked plan + test plan                       │
  └────────────────────┬────────────────────────────────────┘
                       │
                       ▼
  ┌─────────────────────────────────────────────────────────┐
  │ BUILD                                                    │
  │   Your coding agent implements the plan                  │
  │   Plan's test matrix becomes real tests                  │
  │   Confusion Protocol: stop and ask, don't guess          │
  └────────────────────┬────────────────────────────────────┘
                       │
                       ▼
  ┌─────────────────────────────────────────────────────────┐
  │ REVIEW                                                   │
  │   /review   (structural bugs, fix-first)                 │
  │   /codex    (optional second-model review)               │
  └────────────────────┬────────────────────────────────────┘
                       │
                       ▼
  ┌─────────────────────────────────────────────────────────┐
  │ TEST                                                     │
  │   /qa   (real browser, atomic fix commits,               │
  │          auto-generated regression tests)                │
  └────────────────────┬────────────────────────────────────┘
                       │
                       ▼
  ┌─────────────────────────────────────────────────────────┐
  │ SHIP                                                     │
  │   /ship           (sync, test, coverage audit, PR)       │
  │   /land-and-deploy (merge, deploy, verify)               │
  │   /canary         (post-deploy monitoring)               │
  │   /benchmark      (perf regression tracking)             │
  │   /document-release (doc sync)                           │
  └────────────────────┬────────────────────────────────────┘
                       │
                       ▼
  ┌─────────────────────────────────────────────────────────┐
  │ REFLECT                                                  │
  │   /retro   (weekly engineering retro with metrics)       │
  │   /learn   (institutional memory across sessions)        │
  └─────────────────────────────────────────────────────────┘
```

## The artifacts carry context

Every phase writes something the next phase reads. This is not bookkeeping. It is the reason the process works.

| Phase → next | Artifact | Why it matters |
|--------------|----------|----------------|
| Think → Plan | Design doc in `~/.gstack/projects/` | CEO review reads the original problem, not a summary |
| Plan → Build | Locked plan with test matrix | Implementation knows what to build AND what to test |
| Plan → Test | Test plan written by `/plan-eng-review` | `/qa` picks it up automatically — no manual hand-off |
| Review → Ship | Review Readiness Dashboard | `/ship` checks if Eng Review ran before opening PR |
| Test → Ship | Regression tests, health score | PR body shows test delta and QA results |
| Ship → Reflect | Commit log, coverage audit, canary data | `/retro` computes metrics from real output |
| Reflect → Think | Learnings journal | Next `/office-hours` session already knows your patterns |

### The Review Readiness Dashboard

Every review logs its result to a per-branch dashboard:

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

Eng Review is the only required gate (configurable). CEO and Design are informational — recommended for product and UI changes respectively. `/ship` checks the dashboard before creating the PR. If the Eng Review is missing, it asks. Decisions are saved per-branch so you're never re-asked.

## Which reviews for which kind of work

| Building for…                               | Plan stage (before code)     | Live audit (after shipping) |
|---------------------------------------------|------------------------------|-----------------------------|
| **End users** (UI, web, mobile)             | `/plan-design-review`        | `/design-review`            |
| **Developers** (API, CLI, SDK, docs)        | `/plan-devex-review`         | `/devex-review`             |
| **Architecture** (data flow, perf, tests)   | `/plan-eng-review`           | `/review`                   |
| **All of the above**                        | `/autoplan` (runs CEO → design → eng → DX, auto-detects which apply) | — |

The routing isn't arbitrary. Different surface areas fail in different ways. A backend bug fix doesn't need a designer's eye. A new marketing page doesn't need an SRE. Running every review on every change wastes time; running the wrong review wastes trust.

## The four failure modes this prevents

Andrej Karpathy's [AI coding rules](https://github.com/forrestchang/andrej-karpathy-skills) name four failure modes that destroy AI-assisted sprints:

1. **Wrong assumptions** — the agent guesses at an ambiguous requirement and builds the wrong thing.
2. **Overcomplexity** — the agent produces a 200-line abstraction where 10 lines would do.
3. **Orthogonal edits** — the agent "fixes" unrelated code on the way past.
4. **Imperative over declarative** — the agent writes step-by-step logic where state-based declarations would be clearer.

The sprint process defends against all four:

- **`/office-hours`** forces assumptions into the open before code is written.
- **The Confusion Protocol** stops the agent from guessing on architectural decisions.
- **`/review`** catches unnecessary complexity and drive-by edits.
- **`/ship`** transforms tasks into verifiable goals with test-first execution.

If you already use Karpathy-style CLAUDE.md rules, the sprint process is the workflow enforcement layer that makes them stick across an entire feature, not just a single prompt.

## What to take from this chapter

1. **The sprint is a sequence of cognitive modes**, not a pipeline of tools. Each mode has a different job.
2. **Each phase writes an artifact the next phase reads.** That chain is why the system works — nothing falls through.
3. **Plan is four reviews, not one.** CEO, Eng, Design, DevEx catch different failures.
4. **Review precedes test.** Structural bugs that survive CI get caught by reading the code; visual bugs get caught by looking at the page.
5. **Reflect is not optional.** Without a retro, the same patterns and pitfalls compound silently.

The rest of the book goes phase by phase: what specifically to look for, what questions to ask, and what rubrics to use.

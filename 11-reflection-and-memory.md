# 11 — Reflect: Retros and Memory

> At the end of the week I want to know what actually happened. Not vibes — data.

The two most-skipped phases in AI-era engineering are the Think phase at the start and the Reflect phase at the end. Both are where leverage compounds. Skip reflection and the same mistakes recur silently: test coverage slips, one contributor's PRs keep getting rewritten, a bug class keeps appearing in retros that aren't written.

This chapter covers two complementary mechanisms: **retrospectives** (the weekly look back with real numbers) and **learning memory** (the institutional knowledge that persists across sessions and makes every future sprint smarter).

## Part 1 — Retrospectives

`/retro` analyzes commit history, work patterns, and shipping velocity, then writes a candid retrospective. Team-aware: it gives you the deepest treatment on your own work and breaks down every contributor with specific praise and growth opportunities.

### Why metrics, not vibes

Vibes lie. "I had a great week" feels true when you shipped one high-impact thing, forgot to write tests, and spent two days triaging a bug from a prior week. Metrics force honesty:

- You shipped 32 commits. 41% tests.
- Your test ratio dropped from 52% last week.
- You had one 4-hour session at 10pm and seven micro-sessions spread across mornings.

That's not a great week the same way. The retro is the difference between "I felt productive" and "here's what was actually produced."

### The metrics

| Metric | What it means |
|--------|---------------|
| **Features shipped** | From CHANGELOG + merged PR titles |
| **Weighted commits** | Commits × avg files-touched (capped at 20) |
| **Logical SLOC added** | Non-blank, non-comment (primary volume metric) |
| **Test LOC ratio** | Percentage of insertions that are tests |
| **Active days** | Days with 1+ commit |
| **Detected sessions** | Groups of commits with <45min gaps |
| **Avg raw LOC / session-hour** | Real productivity metric |
| **Greptile signal** | (fixes + already-fixed) / (fixes + already-fixed + FPs) |
| **Focus score** | % of commits to the single most-changed top-level directory |

Most of these are obvious in isolation. They become powerful when tracked week over week — you see the trend lines, not just the snapshot.

### Session detection

Commits are grouped by 45-minute gaps. Below 45 minutes between commits = same session. Sessions classify as:

- **Deep sessions** — 50+ minutes.
- **Medium sessions** — 20-50 minutes.
- **Micro sessions** — under 20 minutes (fire-and-forget).

A week of all micro sessions looks like focus but is usually scattered. A week with 3-4 deep sessions and a handful of micros is the rhythm of shipping real work.

### Shipping streak detection

Count consecutive days with 1+ commit to `origin/<default>` going back from today.

- **Team streak** — any contributor.
- **Personal streak** — the current user only.

Streaks are the single most motivating metric in retros because they're binary and visible. Breaking a 47-day streak by missing a Saturday hurts more than it should, which is the point.

### Team breakdown pattern

For each teammate, the retro produces three things:

1. **What they shipped.** 2-3 sentences on contributions, areas, patterns.
2. **Praise (1-2 specific things).** Anchored in actual commits, not "great work." Examples:
   - *"Shipped entire auth module rewrite in 3 focused sessions, 45% test coverage."*
   - *"Every PR under 200 LOC — textbook decomposition."*
   - *"Fixed the N+1 query causing 2s load times."*
3. **Opportunity for growth (1 specific thing).** Framed as leveling up, not criticism:
   - *"Test ratio 12% this week — the payment module is worth investing in before next feature."*
   - *"Most commits in a single burst — spacing across the day reduces context fatigue."*
   - *"All commits 1-4am — sustainable pace matters long-term."*

The tone is what a good manager says in a 1:1. Specific, anchored, encouraging, candid. Never "great job." Never comparing teammates negatively.

### Test health tracking

Every retro tracks:

- Total test files.
- Tests added this period.
- Regression test commits (`test(qa):`, `test(design):`, `test: coverage`).
- Trend delta vs. prior periods.

**If test ratio drops below 20%, flag it as a growth area.** The note: *"100% coverage is the goal. Tests make vibe coding safe."*

### Hotspot analysis

Top 10 most-changed files in the period. Flag:

- Files changed 5+ times — churn hotspots. Usually a sign the architecture in that file is wrong, not that the code is active.
- Test vs production file ratio in hotspots. If only production files churn and no tests, tests aren't being written alongside.
- VERSION / CHANGELOG frequency — version discipline check.

### Commit type breakdown

Categorized by conventional prefix (`feat`, `fix`, `refactor`, `test`, `chore`, `docs`).

**If fix ratio exceeds 50%**, flag it. Shipping fast and fixing fast can look like velocity but often indicates a review gap — code goes in with bugs and comes back out as a fix commit the next day.

### PR size distribution

- **Small** (< 100 LOC)
- **Medium** (100-500 LOC)
- **Large** (500-1500 LOC)
- **XL** (1500+ LOC)

A healthy distribution is mostly S/M with occasional L for real features. Frequent XL usually means scope is too big and review is superficial.

### Eureka moments (optional)

If a eureka journal exists (`~/.gstack/analytics/eureka.jsonl` in gstack's case), filter by the time window and show each insight with its skill, branch, and one-line note. These are the first-principles observations from chapter 1 that got logged when they happened. Retros are where they surface for reinforcement.

### Example output

```
Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests,
12 PRs, peak: 10pm  |  Streak: 47d

## Your Week
32 commits, +2.4k LOC, 41% tests. Peak hours: 9-11pm.
Biggest ship: cookie import system (browser decryption + picker UI).
What you did well: shipped a complete feature with encryption, UI,
and 18 unit tests in one focused push...

## Team Breakdown

### Alice
12 commits focused on app/services/. Every PR under 200 LOC — disciplined.
Opportunity: test ratio at 12% — worth investing before payment gets
more complex.

### Bob
3 commits — fixed the N+1 query on dashboard. Small but high-impact.
Opportunity: only 1 active day this week — check if blocked on anything.

[Top 3 team wins, 3 things to improve, 3 habits for next week]
```

The retro saves a JSON snapshot so next week's run can show trends.

### Global retro

`/retro global` runs the same analysis across all your projects and AI tools (Claude Code, Codex, Gemini). Useful when you're running 10-15 parallel sprints across multiple repos and want to see the forest instead of the trees.

### Tone principles

- **Encouraging but candid.** Not cheerleading; not discouraging.
- **Specific, anchored in commits.** Never "great job." Always "shipped X, which unblocks Y."
- **Growth as investment.** Not criticism.
- **Avoid negative comparisons.** Don't stack teammates against each other.

---

## Part 2 — Learning memory

Retros capture what happened this week. Learning memory captures what the project taught you, **permanently**.

### The concept

Patterns, pitfalls, preferences, architectural decisions accumulate in a project-specific journal. Each learning is a structured entry:

- **Key** (2-5 words): a short identifier.
- **Insight** (one sentence): the learning.
- **Type**: `pattern`, `pitfall`, `preference`, `architecture`.
- **Confidence** (1-10): how strongly supported.
- **Source**: which skill or session surfaced it.

The journal is append-only, stored per-project at `~/.gstack/projects/$SLUG/learnings.jsonl`.

### Why per-project

Different codebases have different conventions. "Use TypeScript strict mode" is a project-level preference. "Prefer `async/await` over Promises" might be a team convention. A universal memory doesn't capture that; a per-project memory does.

### What accumulates

Examples from real gstack projects:

| Type | Insight | Confidence |
|------|---------|------------|
| Pattern | API responses always wrapped in `{ data, error }` envelope | 9/10 |
| Pattern | Tests use factory helpers in `test/support/factories.ts` | 8/10 |
| Architecture | All DB queries go through repository pattern, never direct | 8/10 |
| Pitfall | Webhook handlers must verify signature in middleware, not controller | 9/10 |
| Preference | Prefer Tailwind over styled-components for new components | 7/10 |
| Pitfall | Never call `fetch` in `useEffect` without AbortController | 8/10 |

These aren't guidelines in a wiki. They're auto-surfaced by every skill before it makes a recommendation.

### Confidence scoring

Higher confidence when:

- Multiple independent sessions found the same issue.
- The issue appeared in a late-phase review (higher fidelity than early speculation).
- The issue blocked shipping (the highest-stakes signal).

Lower confidence when:

- Single-session observation.
- Contradicted by other learnings.
- References files that have since changed substantially.

### Pruning stale learnings

Over time, the codebase changes. Learnings that reference deleted files become stale. The pruning flow:

1. For each learning, check if the files it references still exist.
2. Flag as **STALE** if referenced files are gone.
3. Flag as **CONFLICT** if the same key has opposite insights (suggests a past learning was wrong).
4. Ask the user to remove, keep, or update each flagged entry.
5. Updates are append-only — latest entry wins, but old entries stay for history.

### The export

Learnings can be exported to a markdown format suitable for `CLAUDE.md`. Grouped by type (Patterns, Pitfalls, Preferences, Architecture). Confidence scores included. Useful for:

- Onboarding a new team member.
- Sharing context across projects that use similar patterns.
- Making an implicit team convention explicit.

### Where learnings come from

Every skill can write learnings. Common sources:

- **`/review`** — catches the same class of bug twice, logs it as a pitfall.
- **`/investigate`** — traces a root cause, logs the architectural observation.
- **`/plan-eng-review`** — identifies a pattern used consistently, logs it as an architecture decision.
- **`/qa`** — finds a visual bug class that keeps recurring, logs the pitfall.
- **User explicit** — *"remember this for next time"* adds a learning directly.

### How learnings get used

When a skill is about to make a recommendation, it first searches learnings for relevant entries. If it finds one, it displays **"Prior learning applied"** and uses the pattern instead of reinventing.

Example:

```
Design review running...

[Prior learning applied — 9/10]
API responses always wrapped in { data, error } envelope.
All new endpoint proposals will follow this pattern.
```

This is where the compound returns happen. Each learning reduces the number of decisions the next sprint has to make from scratch.

---

## The reflection loop as a whole

```
┌─────────────────────────────────────────────────┐
│  This week's commits, PRs, tests, ships, bugs    │
└──────────────────────┬──────────────────────────┘
                       │
         ┌─────────────▼─────────────┐
         │  /retro                    │
         │  metrics, streaks, per-   │
         │  person breakdowns, test  │
         │  health, hotspots         │
         └─────────────┬─────────────┘
                       │
                       ├── insights feed forward
                       │
         ┌─────────────▼─────────────┐
         │  /learn                    │
         │  append new patterns,      │
         │  prune stale ones, update  │
         │  confidence scores         │
         └─────────────┬─────────────┘
                       │
                       ▼
         ┌───────────────────────────┐
         │  Next sprint's skills      │
         │  auto-apply learnings      │
         │  before recommending       │
         └───────────────────────────┘
```

The loop is what turns individual sprints into a compounding practice. Without it, you run `/office-hours` the same way every time, make the same scope decisions, and hit the same bugs. With it, the tenth sprint is materially smarter than the first.

---

## What to take from this chapter

1. **Retros use data, not vibes.** Metrics force honesty about what actually shipped.
2. **Session detection reveals rhythm.** Deep sessions vs. scattered micros tell different stories.
3. **Shipping streaks motivate.** Binary, visible, hard to fake.
4. **Team breakdowns are 1:1 tone.** Specific praise + specific growth. Never "great job." Never negative comparisons.
5. **Test ratio under 20% is a red flag.** Vibe coding without tests = confident garbage.
6. **Hotspots reveal architecture problems.** Files that churn 5+ times are probably wrong, not active.
7. **Learning memory is per-project.** Each codebase has its own patterns, pitfalls, preferences.
8. **Confidence scoring matters.** Multiple sessions + late-phase review + shipping-blocker = high confidence.
9. **Prune stale learnings.** Deleted files, conflicting insights — flag and resolve.
10. **Compound returns.** Retros catch this week. Memory carries forward. The tenth sprint is smarter than the first.

The final chapter covers two things the sprint phases don't: what to do when work is risky (safety guardrails) and how to scale the whole methodology to ten or fifteen concurrent sprints.

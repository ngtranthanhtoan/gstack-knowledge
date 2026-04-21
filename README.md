# The gstack Knowledge Book

An extraction of the methodology, philosophy, and craft knowledge embedded in [gstack](https://github.com/garrytan/gstack), rewritten as human-readable prose instead of LLM skill instructions.

gstack is Garry Tan's (YC President) open-source "virtual engineering team" — a collection of ~38 Claude Code skills covering every phase of shipping software with AI. The skills are dense with domain knowledge about how to think, plan, build, review, test, ship, and reflect when AI does most of the typing.

This book lifts that knowledge out of the slash-command format so you can read it, argue with it, and apply it — even if you never touch gstack.

## How to read

Each chapter is self-contained but builds on the last. If you only read three, read **01, 02, and 10** — the philosophy, the sprint shape, and how AI changes the economics of completeness.

| # | Chapter | What it covers |
|---|---------|----------------|
| [01](01-the-builder-ethos.md) | **The Builder Ethos** | The Golden Age, Boil the Lake, Search Before Building, User Sovereignty, three layers of knowledge, Build for Yourself |
| [02](02-the-sprint-process.md) | **The Sprint Process** | Think → Plan → Build → Review → Test → Ship → Reflect — why the sequence matters and how each phase feeds the next |
| [03](03-the-think-phase.md) | **Think: YC Office Hours** | Six forcing questions, the reframe pattern, premise challenge, startup mode vs. builder mode, the design doc |
| [04](04-the-plan-phase.md) | **Plan: Four Specialist Reviews** | CEO (scope modes), Eng (architecture), Design (seven passes), DevEx (TTHW, error tiers); autoplan's six decision principles |
| [05](05-design-methodology.md) | **Design End-to-End** | Design system consultation, shotgun exploration, Pretext HTML, live-site audits, and the AI Slop Canon |
| [06](06-the-review-phase.md) | **Review: The Paranoid Staff Engineer** | Structural bugs tests miss, fix-first pattern, completeness gaps, multi-model second opinion |
| [07](07-investigation.md) | **Investigation: The Iron Law** | Root cause before fix, pattern matching, hypothesis testing, the 3-strike rule, structured debug reports |
| [08](08-qa-and-testing.md) | **QA: Giving the Agent Eyes** | Real-browser testing, diff-aware mode, regression test generation, WTF-likelihood self-regulation |
| [09](09-security-audit.md) | **Security: OWASP + STRIDE** | Phase-by-phase attack-surface audit, the 17 false-positive exclusions, confidence gates, active verification |
| [10](10-shipping-and-release.md) | **Ship: The Final Mile** | Test bootstrap, coverage audit, release readiness, land-and-deploy, canary monitoring, performance benchmarks, docs sync |
| [11](11-reflection-and-memory.md) | **Reflect: Retros and Learning** | Shipping streaks, session detection, team breakdowns, institutional memory, pruning stale learnings |
| [12](12-safety-and-parallel-work.md) | **Safety and Parallel Sprints** | Dangerous-command guardrails, freeze boundaries, running 10-15 agents at once, cross-agent coordination |

## Source and staying in sync

[checkpoint.txt](checkpoint.txt) records the last time the upstream `../gstack/` repo was pulled. That timestamp is the baseline for all the knowledge in this book. When gstack is pulled again, [UPDATE_PROCESS.md](UPDATE_PROCESS.md) explains how an AI session should compute the diff and fold new knowledge into the existing chapters.

## Source attribution

Everything here is derived from the gstack repository at `/Users/toannguyen/Repo-Explorer/AI-Development/gstack`:

- `ETHOS.md` — builder philosophy
- `ARCHITECTURE.md` — system internals (chapter 12 borrows from this)
- `README.md` — product framing, the 60-day story
- `docs/skills.md` — human-readable skill deep-dives
- `docs/ON_THE_LOC_CONTROVERSY.md` — measuring AI-era productivity
- ~20 `*/SKILL.md.tmpl` files — the methodology embedded in each skill

Where a skill adds specific rules, rubrics, or checklists, this book preserves them verbatim so you can use them as working artifacts.

## A note on framing

The gstack skills are written as instructions *to an AI agent*. This book translates them back into the form of instructions *to a human collaborator*. Occasionally that means dropping agent-only details (like ref systems, tool calls, template generators) and keeping the craft underneath. Where the plumbing itself is the point — as with the browser daemon or the ref system — it lives in the architecture chapter.

The voice is gstack's voice. I kept Garry's directness where it's load-bearing. This is a knowledge book, not a neutralized summary.
# gstack-knowledge

# 01 — The Builder Ethos

> "I don't think I've typed like a line of code probably since December, basically, which is an extremely large change." — Andrej Karpathy

Before any of the methodology in this book makes sense, you need the worldview underneath it. gstack exists because four things are simultaneously true in 2026 that weren't true in 2013, and the consequences touch every engineering decision downstream.

## The Golden Age

A single person with AI can now build what used to take a team of twenty. The engineering barrier is gone. What remains is **taste, judgment, and the willingness to do the complete thing**.

This is not a prediction. It is happening right now. 10,000+ usable lines of code per day, 100+ commits per week, shipped by one person, part-time, using the right tools. The compression ratio between human-team time and AI-assisted time runs from 3× at the low end to 100× at the high end:

| Task type                  | Human team | AI-assisted | Compression |
|----------------------------|-----------:|------------:|------------:|
| Boilerplate / scaffolding  | 2 days     | 15 min      | ~100×       |
| Test writing               | 1 day      | 15 min      | ~50×        |
| Feature implementation     | 1 week     | 30 min      | ~30×        |
| Bug fix + regression test  | 4 hours    | 15 min      | ~20×        |
| Architecture / design      | 2 days     | 4 hours     | ~5×         |
| Research / exploration     | 1 day      | 3 hours     | ~3×         |

This table changes every build-vs-skip decision you used to make on autopilot. The last 10% of completeness that teams used to skip? It costs seconds now.

## 1. Boil the Lake

**When the complete implementation costs minutes more than the shortcut — do the complete thing. Every time.**

The old habit was to ask: "what's the MVP? which corners can we cut? what can we defer to a follow-up PR?" That habit was correct when human engineering time was the bottleneck. It is wrong now.

### Lakes vs. oceans

A **lake** is boilable: 100% test coverage for a module, a full feature implementation, all the edge cases, complete error paths, a matching migration script, updated docs. A lake has a finite surface area. AI can boil it in an hour.

An **ocean** is not boilable: rewriting an entire system from scratch, multi-quarter platform migrations, crossing N service boundaries to normalize a data model. Oceans are still oceans.

The rule: **boil lakes, flag oceans as out of scope**.

### The 70-line delta

When evaluating "approach A (full, ~150 LOC) vs approach B (90%, ~80 LOC)" — always prefer A. The 70-line delta costs seconds with AI coding. "Ship the shortcut" is legacy thinking from when humans were the bottleneck.

### Anti-patterns to notice

- "Choose B — it covers 90% with less code." (If A is 70 lines more, choose A.)
- "Let's defer tests to a follow-up PR." (Tests are the cheapest lake to boil.)
- "This would take 2 weeks." (Say: "2 weeks human / ~1 hour AI-assisted.")

When you're estimating effort, show both sides. A human-only estimate shoves you back into 2013's tradeoffs.

## 2. Search Before Building

**The 1000× engineer's first instinct is "has someone already solved this?" not "let me design it from scratch."**

The cost of searching is near-zero. The cost of not searching is reinventing something worse. Before building anything involving unfamiliar patterns, infrastructure, concurrency, or runtime capabilities — stop and search first.

### The three layers of knowledge

There are three distinct sources of truth when building anything. The best engineers operate fluently in all three. The worst ones pick one and stay there.

**Layer 1 — Tried and true.** Standard patterns, battle-tested approaches, things deeply in distribution: SQL with indexes, HTTP caching, ACID transactions, the runtime's built-in concurrency primitive. You probably already know these. The risk is not that you don't know — it's that you assume the obvious answer is right when occasionally it isn't. The cost of checking is near-zero. And once in a while, questioning the tried-and-true is exactly where brilliance occurs.

**Layer 2 — New and popular.** Current best practices, blog posts, ecosystem trends, this year's framework. Search for these. But scrutinize what you find: humans are subject to mania. Mr. Market is either too fearful or too greedy. The crowd can be wrong about new things as easily as old things. Search results are inputs to your thinking, not answers.

**Layer 3 — First principles.** Original observations derived from reasoning about the specific problem at hand. These are the most valuable of all. Prize them above everything else. The best projects avoid mistakes (don't reinvent the wheel — Layer 1) while also making brilliant observations that are out of distribution (Layer 3).

### The Eureka Moment

The most valuable outcome of searching is *not* finding a solution to copy. It is:

1. Understanding what everyone is doing and *why* (Layers 1 + 2)
2. Applying first-principles reasoning to their assumptions (Layer 3)
3. Discovering a clear reason why the conventional approach is wrong

This is the 11 out of 10. The truly superlative projects are full of these moments — zig while others zag. When you find one, **name it. Celebrate it. Build on it.**

### Anti-patterns

- Rolling a custom solution when the runtime has a built-in. (Layer 1 miss.)
- Accepting blog posts uncritically in novel territory. (Layer 2 mania.)
- Assuming tried-and-true is right without questioning premises. (Layer 3 blindness.)

### The search protocol

Before designing anything non-trivial:

1. Search for `"{runtime} {thing} built-in"`.
2. Search for `"{thing} best practice {current year}"`.
3. Check official runtime/framework docs.

If all three layers come back with the same answer, that's Layer 1 — use it. If they diverge, that's usually where the real problem lives.

## 3. User Sovereignty

**AI models recommend. Users decide. This is the one rule that overrides all others.**

Two AI models agreeing on a change is a strong signal. It is *not* a mandate. The user always has context the models lack: domain knowledge, business relationships, strategic timing, personal taste, future plans that haven't been shared yet.

When Claude and Codex both say "merge these two things" and the user says "no, keep them separate" — **the user is right. Always.** Even when the models can construct a compelling argument for why the merge is better.

### The philosophical lineage

Andrej Karpathy calls this the "Iron Man suit" philosophy: great AI products augment the user, not replace them. The human stays at the center.

Simon Willison warns that "agents are merchants of complexity" — when humans remove themselves from the loop, they don't know what's happening.

Anthropic's own research shows that experienced users interrupt Claude *more* often, not less. Expertise makes you more hands-on, not less.

### The generation-verification loop

The correct pattern:

- AI **generates** recommendations.
- The user **verifies** and decides.
- The AI never skips the verification step because it's confident.

### The rule

When you and another model agree on something that changes the user's stated direction — **present the recommendation, explain why you both think it's better, state what context you might be missing, and ask. Never act.**

### Anti-patterns

- "The outside voice is right, so I'll incorporate it." (No. Present it. Ask.)
- "Both models agree, so this must be correct." (Agreement is signal, not proof.)
- "I'll make the change and tell the user afterward." (Ask first. Always.)
- Framing an AI assessment as settled fact in a "My Assessment" column. (Present both sides. Let the user fill in the assessment.)

## 4. Build for Yourself

The best tools solve your own problem. gstack exists because its creator wanted it. Every feature was built because it was *needed*, not because it was requested.

If you're building something for yourself, trust that instinct. The specificity of a real problem beats the generality of a hypothetical one every time. You already have a user: you. You already have a feedback loop: your own frustration. That's two things most products never get.

## How these principles work together

- **Boil the Lake** says: do the complete thing.
- **Search Before Building** says: know what exists before you decide what to build.
- **User Sovereignty** says: recommend, don't impose.
- **Build for Yourself** says: the best problem is one you personally feel.

Together: search first, then build the complete version of the right thing, recommend don't impose, and start from a problem you actually have.

**The worst outcome** is building a complete version of something that already exists as a one-liner.

**The best outcome** is building a complete version of something nobody has thought of yet — because you searched, understood the landscape, and saw what everyone else missed.

---

## The evidence it's already happening

The LOC debate is not really about LOC. It's about whether the ground under software engineering moved. Here's the raw data from Garry Tan's own repos, 2013 vs 2026 (through day 108 of 2026):

|                  | 2013 (full year) | 2026 (108 days) | Multiple |
|------------------|----------------:|----------------:|---------:|
| Logical SLOC     |           5,143 |       1,233,062 |     240× |
| Logical SLOC/day |              14 |          11,417 |     810× |
| Commits          |              71 |             351 |     4.9× |
| Files touched    |             290 |          13,629 |      47× |
| Active repos     |               4 |              15 |    3.75× |

Logical SLOC strips blank lines, comments, and whitespace — fluff that AI generates more of. Apply a second aggressive deflation for AI verbosity (2×, the upper bound a skeptic would demand):

- Raw 2026 per-day rate: **11,417**
- With 2× AI-verbosity deflation: **5,708**
- Multiple on daily pace with both deflations: **408×**

Pick any deflation coefficient you like:

- 2× (aggressive): **408×**
- 5× (unfounded): **162×**
- 10× (pathological): **81×**
- 100× (impossible — one line per minute sustained): **8×**

**The argument about the size of the coefficient doesn't change the conclusion.** The number is large regardless.

### Tests are what make it real

Early 2026 Garry was shipping fast without tests and "getting destroyed in bug land." The shift: 30% test-to-code ratio → 100% coverage on critical paths → 2,000+ tests across repos.

> Testing at multiple levels is what makes AI-assisted coding actually work. Unit tests, E2E tests, LLM-as-judge evals, smoke tests, slop scans. Without those layers, you're just generating confident garbage at high speed. With them, you have a verification loop that lets the AI iterate until the code is actually correct.

This is why the rest of this book spends so much time on review, QA, and shipping discipline. The velocity isn't free. It's bought with testing layers that used to be too expensive to justify.

### What about quality?

The honest metrics from the same 15 active repos:

- **Revert rate:** 7 / 351 commits = **2.0%** (mature OSS codebases run 1-3%)
- **Post-merge fix rate:** 22 / 351 commits = **6.3%** (healthy — zero would mean not catching own mistakes)
- **Slop score:** 62% reduction in one refactoring session after a third party measured it

### The real metric

> Time to first user is the metric that matters, not LOC. The 60-day cycle from "I wish this existed" to "it exists and someone is using it" is the real shift.

The gap between "I want this tool" and "this tool exists and I'm using it" collapsed from 3 weeks to 3 hours. LOC is downstream evidence. Products-per-quarter and features-per-week went up by a similar multiple.

---

## What to take from this chapter

1. **Completeness is now cheap.** If the complete version is a lake, boil it. Quote both human time and AI-assisted time when you estimate.
2. **Search first.** Three layers: tried-and-true, new-and-popular, first-principles. Name your eureka moments.
3. **Recommend, don't decide.** Agreement between models is signal, not authority. The user has context you don't.
4. **Solve your own problem.** The best tools are ones the author uses every day.
5. **Tests are the unlock.** Speed without a verification layer is confident garbage at scale. Speed *with* a verification layer is the ground moving.

The rest of the book is about how to operationalize this — one phase of the sprint at a time.

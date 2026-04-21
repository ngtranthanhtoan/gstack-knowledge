# 04 — Plan: Four Specialist Reviews

Once you know what you're building, the plan phase locks it down. Four different specialists take turns with the design doc, each looking for a different class of failure. Run them in sequence (CEO → Design → Eng → DevEx) or let `/autoplan` run them with encoded decision principles.

Each review is opinionated by design. They disagree with each other sometimes. That's fine — the user is the final decider.

## Why four reviews?

Because a team has at least four heads. In a well-run startup:

- The CEO asks *what* is being built and *why* it matters.
- The eng manager asks *can this be built, and what breaks?*
- The designer asks *does this feel like a real product, or does it look like every other AI site?*
- The DX lead asks *will a developer actually get value from this in the first five minutes?*

Each question surfaces different failures. Merging them all into one "general review" means each question gets 25% attention, and the weakest 25% is where the bugs live.

## The CEO / Founder review

**Specialist:** The founder-mode of your own thinking. Brian Chesky mode.
**Job:** Not implementing the obvious ticket. Asking what the 10-star product hiding inside the request actually is.

### The classic example

You're building a Craigslist-style listing app. The user says:

> *"Let sellers upload a photo for their item."*

A weak assistant adds a file picker and saves an image. That is not the real product.

CEO review asks whether "photo upload" is even the feature. The real feature is probably **helping someone create a listing that actually sells**. If that's the job, the whole plan changes. Now the questions become:

- Can we identify the product from the photo?
- Can we infer the SKU or model number?
- Can we search the web and draft the title and description automatically?
- Can we pull specs, category, and pricing comps?
- Can we suggest which photo will convert best as the hero image?
- Can we detect when the uploaded photo is ugly, dark, cluttered, or low-trust?
- Can we make the experience feel premium instead of like a dead form from 2007?

CEO review asks **"what is the 10-star product hiding inside this request?"** not "how do I add this feature?"

### Four scope modes

You pick one. The mode commits fully — no silent drift.

1. **SCOPE EXPANSION.** Dream big. Propose the ambitious version. Every expansion is presented as an individual decision the user opts into. Recommendations are enthusiastic.
2. **SELECTIVE EXPANSION.** Hold current scope as baseline, but surface opportunities one at a time with neutral recommendations. The user cherry-picks what's worth doing.
3. **HOLD SCOPE.** Current scope accepted. Maximum rigor on execution. Catch every failure mode, edge case, observability gap.
4. **SCOPE REDUCTION.** Strip to minimum viable. What's the absolute smallest version that delivers core value?

Visions and decisions persist to `~/.gstack/projects/` so they survive beyond the conversation. Exceptional ones can be promoted to `docs/designs/` in the repo for the team.

### The 11 review sections

Not all apply equally to every change — but they're run mandatorily, and "N/A" is a decision you make, not a default.

1. **Architecture + dependency graph + data flows.** Four paths for each flow: happy, nil, empty, error.
2. **Error & rescue map.** Name every exception. Trace the rescue action. State the user impact.
3. **Security & threat model.** Attack vectors, input validation, auth, injection.
4. **Data flow & interaction edge cases.** Double-click, stale state, slow networks, offline.
5. **Code quality.** DRY, naming, patterns, complexity.
6. **Test coverage diagram.** New UX flows, data flows, codepaths, async work, integrations, error paths — each must have a test type assigned.
7. **Performance.** N+1 queries, memory, indexes, caching, slow paths.
8. **Observability.** Logging, metrics, tracing, alerts, dashboards, runbooks.
9. **Deployment & rollout.** Migration safety, feature flags, rollback, deploy-time risks.
10. **Long-term trajectory.** Technical debt, reversibility, knowledge concentration, ecosystem fit.
11. **Design & UX** (if UI scope detected). Information architecture, interaction states, user journey, empty states, accessibility.

### The failure modes registry

Every codepath gets mapped:

| Codepath | Failure mode | Rescued? | Tested? | User-visible? |

Silent + untested = **CRITICAL GAP**. This is the single most useful output of CEO review. Most production incidents trace back to a codepath on this grid where "Rescued?" was "No" and "Tested?" was "No."

### Principles

- **Completeness is cheap with AI.** If approach A is full (150 LOC) and approach B is 90% (80 LOC), prefer A. AI cost is seconds, not days. Boil the lake.
- **Implementation alternatives are mandatory.** Always produce 2-3 approaches before selecting a mode. Don't default to minimal just because it's smaller.
- **Zero silent failures.** Every failure mode must be visible. Catch-all error handling is a code smell.
- **Design for the 6-month future, not just today.** "Scrap it and do this instead" is on the table if a fundamentally better approach exists.

---

## The Engineering Manager review

**Specialist:** Your best technical lead.
**Job:** Make the idea **buildable**. Lock in architecture, data flow, diagrams, edge cases, and tests.

### The mode shift

CEO mode is expansive. Eng mode is different intelligence entirely. Not "wouldn't it be cool if." Not more sprawling ideation. Architecture. System boundaries. State transitions. Trust boundaries. Failure modes. Test coverage.

### Diagrams force hidden assumptions into the open

This is the single biggest unlock:

> LLMs get way more complete when you force them to draw the system. Sequence diagrams, state diagrams, component diagrams, data-flow diagrams, even test matrices. Diagrams force hidden assumptions into the open. They make hand-wavy planning much harder.

ASCII diagrams in the plan are not decoration. They are proof that every box is understood, every arrow has a direction, every state has a transition, and nothing is "TBD."

### The listing-app example (continued)

Take the smart listing flow from CEO review: uploads photos → identifies product → enriches listing → drafts title/description → picks hero image.

Eng review asks:

- What's the architecture for upload, classification, enrichment, and draft generation?
- Which steps happen synchronously, which go to background jobs?
- Where are the boundaries between app server, object storage, vision model, search/enrichment APIs, and the listing database?
- What happens if upload succeeds but enrichment fails?
- What happens if product identification is low-confidence?
- How do retries work?
- How do we prevent duplicate jobs?
- What gets persisted when? What can be safely recomputed?

Each one forces a diagram. Each diagram forces a decision.

### The scope challenge (Step 0)

Before anything else, check complexity:

- \>8 files?
- \>2 new classes/services?

If yes, that's a smell. Proactively recommend scope reduction before running through the rest of the review.

### Search before building

Before designing any solution that involves concurrency, unfamiliar patterns, infrastructure, or runtime capabilities that might have a built-in:

1. Search for `"{runtime} {thing} built-in"`.
2. Search for `"{thing} best practice {current year}"`.
3. Check official runtime/framework docs.

Layer 1 (tried-and-true) beats Layer 2 (new-popular). Layer 3 (first-principles) trumps both when you catch a wrong assumption in the conventional wisdom.

### Four mandatory sections

1. **Architecture** — system boundaries, dependency graph, data flow bottlenecks, scaling limits, single points of failure, security boundaries, production failure scenarios, rollback procedure. Require ASCII diagrams.
2. **Code Quality** — module structure, DRY violations, error handling, missing edge cases, over/under-engineering, complexity. Update existing ASCII diagrams for accuracy.
3. **Test Coverage** — for every new codepath, UX flow, data flow, async work, integration, error path: what test type covers it? Happy path? Failure path? Edge case? Test pyramid check.
4. **Performance** — N+1 queries, memory limits, database indexes, caching opportunities, slow paths p99 latency, connection pool pressure.

### Distribution architecture

If the plan introduces a new artifact (CLI binary, library, container), where's the CI/CD pipeline for building and publishing? Code without distribution is code nobody can use. This is one of the easiest gaps to catch and one of the most often missed.

### TODOS cross-reference

Is this plan blocking deferred items? Can deferred items be bundled without expanding scope? Should this plan create new TODOs?

### Principles

- **Boring by default.** Choose proven technology. Incremental over revolutionary.
- **Reversibility preferred.** Design for tired humans at 3am, not best engineers on best day.
- **DX is product quality.** Slow CI or painful deploys breed worse software and higher attrition.
- **Worktree parallelization.** Analyze implementation steps for parallel execution. Module-level dependencies (not file-level). Identify conflicts before they happen.

### Plan-to-QA flow

When Eng review finishes the test coverage section, it writes a test plan artifact. When `/qa` runs later, it picks up that artifact automatically — engineering review feeds directly into QA testing with no manual copy-paste.

---

## The Design review (before code)

**Specialist:** Senior designer reviewing the plan before you write a single line of code.
**Job:** Catch the 80% of design problems that are cheap to fix in the plan and expensive to fix after implementation.

Most plans describe what the backend does but never specify what the user actually sees. Empty states? Error states? Loading states? Mobile layout? AI slop risk? These decisions get deferred to "figure it out during implementation" — and an engineer ships *"No items found."* as the empty state because nobody specified anything better.

### Rating-driven review

Each of seven dimensions gets rated 0-10. The rating drives the work:

- Rate low = lots of fixes
- Rate high = quick pass

**Don't just rate.** Explain the gap. Show what a 10 looks like. Fix the plan. Re-rate.

### The seven passes

#### Pass 1 — Information Architecture
What does the user see first? Second? Third? Is hierarchy defined? A 10 defines primary/secondary/tertiary content hierarchy for every screen.

#### Pass 2 — Interaction State Coverage
For each feature, five states must be specified: **loading, empty, error, success, partial**. A plan with 4 features × 5 states = 20 state specifications. Count how many are actually in the plan. That ratio is your rating.

#### Pass 3 — User Journey & Emotional Arc
Does the plan consider the *emotional experience*? Storyboard the journey with the mood/feeling at each step. A 10 names the feeling (*anxious, confident, delighted, bored, relieved*) at each key moment.

#### Pass 4 — AI Slop Risk
The big one. Does the plan specify intentional UI or generic patterns?

Patterns that score low:
- "Cards with icons" — what differentiates them?
- "Hero section with gradient" — what makes it THIS product?
- "Clean, modern UI" — no content.
- "Tasteful animations" — meaningless without specifics.

A 10 describes UI with enough specificity that two designers implementing it would produce recognizably similar work. See chapter 5 for the full AI Slop Canon.

#### Pass 5 — Design System Alignment
Does the plan match `DESIGN.md`? Do new components fit the existing vocabulary? A plan that silently introduces a new button style or font family scores low until it justifies why.

#### Pass 6 — Responsive & Accessibility
Mobile/tablet specified (not just "stacked")? Keyboard navigation, screen readers, 44px touch targets, color contrast? A 10 names specific breakpoints and interaction patterns per breakpoint.

#### Pass 7 — Unresolved Decisions
What ambiguities will haunt implementation? What the empty state looks like. What the mobile nav pattern is. What happens at the end of the list. A 10 has zero open design questions.

### Seven design principles

These are the bones. Every pass is checking one of these:

1. **Empty states are features.** Warmth, primary action, context — not *"No items found."*
2. **Every screen has hierarchy.** Respecting user time means making it obvious what matters.
3. **Specificity over vibes.** Name the font, the spacing scale, the interaction pattern.
4. **Edge cases are user experiences.** 47-character names, zero results, first-time vs power user — all are real sessions.
5. **AI slop is the enemy.** Generic card grids, hero sections, every-other-AI-site patterns.
6. **Responsive = intentional per viewport.** Not "stacked on mobile."
7. **Accessibility is not optional.** Keyboard, screen readers, contrast — all tested, not hoped for.

### Subtraction default

From Dieter Rams: *"As little design as possible."* If an element doesn't earn pixels, cut it. Every element either builds or erodes trust.

### Principles

- Design is intentional or it's accidental.
- Invisible = perfect.
- Specificity forces clarity.
- Edge cases are where products fail.

---

## The Developer Experience review

**Specialist:** DX Lead — UX for developers.
**Job:** When you're building an API, CLI, SDK, or docs, figure out if a real developer can get to "oh wow" in five minutes or fewer.

### Three DX modes

- **DX EXPANSION** — competitive advantage. Make the DX a reason to choose this product.
- **DX POLISH** — bulletproof every touchpoint.
- **DX TRIAGE** — critical gaps only. Ship the minimum.

### Five mandatory Step 0 investigations

Before scoring anything:

#### 0A — Developer Persona
Who is the target developer? A YC founder? A platform engineer? A frontend dev? OSS contributor? Student? Different personas have different tolerance and expectations. A 5-minute install is fine for a platform engineer and fatal for a student.

#### 0B — Empathy Narrative
Walk the actual getting-started path. Reference real files. First-person from the persona's perspective: *"I open README. I see [actual heading]. I run [actual command]. I see [actual output]. My reaction: [confused / excited / bored]."*

#### 0C — Competitive Benchmark
What's the **TTHW** (time-to-hello-world) for competitors? Which tier to target?

- **Champion** — under 2 minutes (Stripe, Vercel, Firebase)
- **Competitive** — 2-5 minutes (Docker, most OSS)
- **Current trajectory** — whatever yours is today

Reference points: Stripe 30s, Vercel 2min, Firebase 3min, Docker 5min. Benchmark against competitors, not internal estimates.

#### 0D — Magical Moment Design
What's the instant a developer goes *"oh wow, this is real"*? Four delivery vehicles:

- **Interactive playground** — zero install, try in browser.
- **Copy-paste demo command** — one terminal command produces output.
- **Video/GIF** — passive but zero friction.
- **Guided tutorial** — deepest engagement, longest time-to-magic.

#### 0F — Developer Journey Trace
For each stage (Discover → Install → Hello World → Real Usage → Debug → Upgrade), trace the actual path. Identify friction points. One `AskUserQuestion` per friction point — don't batch them.

### The eight review passes

Each 0-10:

1. **Getting Started Experience** — Zero to hello world in under 5 minutes? One command? Free tier? Sandbox available?
2. **API/CLI/SDK Design** — Intuitive? Consistent? Guessable naming? Sensible defaults? Complete or does the dev drop to raw HTTP for edge cases?
3. **Error Messages** — see error tiers below.
4. **Documentation** — Copy-paste complete examples? Progressive disclosure? Playgrounds? Versioned? Tutorials and references both?
5. **Upgrade & Migration** — Backward compatible? Deprecation warnings? Migration guides? Codemods? Semantic versioning?
6. **Developer Environment** — Language server? CI/CD integration? TypeScript types? Testing support? Hot reload? Cross-platform?
7. **Community & Ecosystem** — Open source? Community channels? Real-world examples? Plugin/extension ecosystem? Contributing guide?
8. **DX Measurement** — TTHW tracked? Journey analytics? Feedback mechanisms? Friction audits planned?

### Error message tiers

This is the single most actionable rubric in DX:

**Tier 1 — Elm-style.** Conversational, first-person, exact location, suggested fix.

```
I cannot find `elm-package.json` in this directory.

I was looking for it here:
    /Users/you/project/elm-package.json

Try running `elm package install` first, or maybe you meant to `cd` somewhere else?
```

**Tier 2 — Rust-style.** Error codes, documentation links, targeted help.

```
error[E0308]: mismatched types
  --> src/main.rs:4:13
   |
4  |     let x: i32 = "hello";
   |                  ^^^^^^^ expected i32, found &str
   |
   = note: for more information, see https://doc.rust-lang.org/error-codes/E0308.html
```

**Tier 3 — Stripe-style.** Structured JSON with type, code, message, param, doc_url.

```json
{
  "error": {
    "type": "invalid_request_error",
    "code": "parameter_missing",
    "message": "Missing required param: customer.",
    "param": "customer",
    "doc_url": "https://stripe.com/docs/error-codes/parameter-missing"
  }
}
```

Your target tier depends on your surface area. CLIs aim for Tier 1. Compilers aim for Tier 2. APIs aim for Tier 3.

### Three DX first principles

1. **Zero friction at T0.** Nothing between the user and their first success.
2. **Learn by doing.** Copy-paste examples must work out of the box.
3. **Fight uncertainty.** Errors must tell the developer what to do next.

### Principle

> Developers are chefs cooking for chefs. The bar is higher because downstream impact is broader.

Tone matters (friendly errors vs generic). Examples must work. Free tier removes friction. If a developer can't get to success in the first five minutes, they're gone.

---

## Autoplan: running all four automatically

Running the four reviews individually can mean answering 15-30 intermediate questions. Each is valuable, but sometimes you want the gauntlet to run without stopping for every decision.

`/autoplan` reads all four review skills and runs them sequentially: CEO → Design → Eng → (DX if applicable). It makes decisions automatically using six encoded principles and surfaces only **taste decisions** for your approval at a final gate.

### The six encoded decision principles

1. **Completeness.** Ship the whole thing. Pick the approach covering more edge cases.
2. **Boil lakes.** Fix everything in blast radius. Auto-approve expansions under 1 day effort in affected files.
3. **Pragmatic.** If two options fix the same thing, pick the cleaner one.
4. **DRY.** Duplicates existing functionality? Reject. Reuse what exists.
5. **Explicit over clever.** 10-line obvious fix beats 200-line abstraction.
6. **Bias toward action.** Merge > review cycles > stalled deliberation. Flag concerns but don't block.

### What auto-resolves

- Mechanical decisions (always run codex? always yes. reduce scope on already-complete plan? always no.)
- Decisions the six principles cover cleanly. Roughly 9 out of 10 issues.

### What gets surfaced as a taste decision

- **Close calls.** Two options score within 1 point on the rubric.
- **Borderline scope expansions.** The principles recommend adding, but it expands scope by 3-5 files.
- **Cross-model disagreements.** When Claude and Codex disagree on a recommendation.
- **User challenges.** When both Claude and Codex think the user should change their stated direction (merge, split, add, remove).

The user's original direction is the default. Models must *make the case* for change. The user remains sovereign.

### Example output

```
Running CEO review... [4 scope decisions auto-resolved]
Running design review... [3 design dimensions auto-scored]
Running eng review... [2 architecture decisions auto-resolved]

TASTE DECISIONS (need your input):
1. Scope: Codex suggested adding search — borderline expansion. Add?
2. Design: Two approaches scored within 1 point. Which feels right?

[Shows both options with context]
```

Two questions instead of thirty. Same coverage.

---

## What to take from this chapter

1. **Plan is not one review. It's four.** Each specialist catches failures the others miss.
2. **CEO review looks for the 10-star product** hiding inside the request. Four scope modes commit you to a direction.
3. **Eng review forces diagrams.** Hand-wavy planning dies on a sequence diagram.
4. **Design review rates seven dimensions 0-10** and catches AI slop patterns before pixels exist.
5. **DevEx review checks TTHW** and benchmarks against real competitors, not internal estimates.
6. **Autoplan runs the gauntlet** with six decision principles and surfaces only genuine taste calls.
7. **Every review writes an artifact** the build and test phases will read. The chain is the feature.

The next chapter zooms in on design — the one craft area where AI fails hardest by default, and the place where gstack's methodology is densest.

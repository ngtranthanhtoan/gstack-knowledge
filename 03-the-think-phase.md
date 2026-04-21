# 03 — Think: YC Office Hours

Before you plan, before you review, before you write a single line of code — sit down with a YC-style partner and think about what you're actually building. Not what you think you're building. What you're **actually** building.

Most projects fail because the wrong thing was built, not because it was built wrong. The Think phase is insurance against the wrong-thing failure mode. Thirty minutes of interrogation now saves three weeks of implementing a feature nobody will use.

## The reframe is the whole point

Here's what happened on a real project. The user said: *"I want to build a daily briefing app for my calendar."* Reasonable request. The assistant asked about the pain — specific examples, not hypotheticals. The user described:

- An executive assistant missing things across multiple Google Calendars
- Events with stale info and wrong locations
- Prep docs that read like AI slop
- Hours wasted tracking down meeting logistics

After listening, the skill came back with:

> *"I'm going to push back on the framing, because I think you've outgrown it. You said 'daily briefing app for multi-Google-Calendar management.' But what you actually described is a personal chief of staff AI."*

Then it extracted **five capabilities the user didn't realize they were describing**:

1. **Watches your calendar** across accounts, detects stale info, missing locations, permission gaps
2. **Generates real prep work** — not logistics summaries, but *the intellectual work* of preparing for a board meeting, a podcast, a fundraiser
3. **Manages your CRM** — who are you meeting, what's the relationship, what do they want, what's the history
4. **Prioritizes your time** — flags when prep needs to start early, blocks time proactively, ranks by importance
5. **Trades money for leverage** — actively looks for ways to delegate or automate

That reframe changed the entire project. The user was about to build a calendar app. Now they were building something ten times more valuable — because the skill listened to their **pain** instead of their **feature request**.

**The lesson:** users describe features; pain reveals products. The Think phase's primary job is to translate one into the other.

## Two modes

### Startup mode

For founders, intrapreneurs, and anyone building a product that needs users. You get **six forcing questions** distilled from how YC partners evaluate products. The questions are uncomfortable on purpose. If you can't name a specific human who needs your product, that's the most important thing to learn before writing any code.

### Builder mode

For hackathons, side projects, open source, learning, and having fun. You get an enthusiastic collaborator who helps you find the coolest version of your idea. What would make someone say "whoa"? What's the fastest path to something you can share? The questions are generative, not interrogative.

The mode isn't arbitrary. Asking "who will pay for this" about a weekend hackathon is obnoxious. Asking "what would make this magical" about a payments platform is malpractice. Pick the one that fits.

---

## The six forcing questions (startup mode)

This is the core of office-hours methodology. Apply them in order. Don't advance until the previous one has a specific, falsifiable answer.

### Q1: Demand Reality

**Push until you hear specific behavior.** Someone paying. Someone expanding usage. Someone panicking if the thing disappeared tomorrow.

**Red flags:**
- "People say it's interesting."
- "500 waitlist signups."
- "VCs are excited."

None of those are demand. Demand is retention, payment, expansion, panic.

**Frame precision:** Define key terms so you could *measure* them. Distinguish real pain (observed) from hypothetical pain (thought experiment). "My assistant spends 6 hours a week on this" is real pain. "I think assistants probably waste time on this" is a thought experiment.

### Q2: Status Quo

**What are users doing right now to solve this problem — even badly?**

Push for: specific workflow, hours wasted, dollars lost, duct-taped tools, people hired to do manual work.

**Red flag:** *"Nothing exists, that's why it's a huge opportunity."*

This almost always means the problem isn't painful enough. When the pain is real, people build spreadsheets, hire VAs, buy adjacent tools and misuse them, or pay their assistant overtime. The absence of a workaround is usually the absence of a problem.

The status quo is your real competitor. Not a competitor with a funded team — a spreadsheet and a Slack channel.

### Q3: Desperate Specificity

**Name the actual human.** Title. What gets them promoted? Fired? Keeps them up at night?

Push for: a name, a role, a consequence they face. Ideally a name you've heard directly from them.

**Red flag:** "Healthcare enterprises." "SMBs." "Marketing teams." These are filters, not people.

If you cannot finish the sentence "*[Name] is a [title] at [company] who [specific problem]*," you don't have a target user. You have a demographic.

### Q4: Narrowest Wedge

**What's the smallest version someone would pay real money for — this week, not after the full platform?**

Push for: one feature, one workflow, one automation they'd pay for or ship in days.

**Red flag:** "We need the full platform first."

This usually signals attachment to architecture, not value. The full platform is a three-month project. The wedge is what you can put in someone's hands tomorrow.

The narrowest wedge of "personal chief of staff AI" is not "the full chief of staff AI." It's "a daily briefing that actually works." You learn from real usage. CRM data comes naturally in week two.

### Q5: Observation & Surprise

**Have you watched someone use this without helping? What surprised you?**

Push for: a surprise. Something the user did that contradicted your assumptions.

**Red flags:**
- "We sent a survey."
- "We did demo calls."
- "Nothing surprising, going as expected."

Surveys lie (people answer what they think you want to hear). Demos are theater (people perform for the maker). "Expected" means you're filtering observations through the story you want to tell.

The single most valuable thing you can do as a product-builder: sit silently while someone uses your thing. Watch what confuses them. Watch what they do wrong. Watch the moments they almost give up. None of that appears in any survey.

### Q6: Future-Fit

**In 3 years, when the world looks different, does your product become more essential or less?**

Push for: a specific claim about how users' world changes.

**Red flag:** Generic rising-tide arguments every competitor can make. ("AI is going to be huge, so our AI product will be huge.")

Good future-fit looks like: *"As more companies adopt hybrid work, the 'who do I talk to about X?' problem gets worse — directories don't work when nobody's at the same desk."* That's a specific claim about how the world changes and how it sharpens your product's thesis.

---

## The reframe pattern

After listening to the pain (Q1-3 provide the material), state back what you think the user is actually building:

> *"Let me restate what I think you're actually building: [reframe]. Does that capture it better?"*

Takes 60 seconds. Saves three weeks if it lands.

Two rules for reframes:

1. **The reframe must come from the user's words, not yours.** If you hallucinate a product they didn't describe, they'll correct you, and you'll lose trust. Pull directly from pain they named.
2. **Name the shift.** "You said X. I heard Y." Make the delta explicit. That's what makes the user realize they've been describing something bigger than what they came in saying.

---

## Premise challenge

After the reframe, turn the product direction into falsifiable premises. Not "does this sound good?" — actual testable claims.

Example, from the chief-of-staff case:

1. The calendar is the anchor data source, but the value is in the intelligence layer on top.
2. The assistant doesn't get replaced — they get superpowered.
3. The narrowest wedge is a daily briefing that actually works.
4. CRM integration is a must-have, not a nice-to-have.

The user agrees, disagrees, or adjusts each one. Every premise they accept becomes **load-bearing** in the design doc — referenced later by CEO review, Eng review, and downstream decisions.

### Premise challenges worth always running

Regardless of the product:

1. **Is this the right problem?** Could a different framing yield a simpler or more impactful solution?
2. **What happens if we do nothing?** Real pain or hypothetical?
3. **What existing code partially solves this?** Don't rebuild what already exists.
4. **Distribution blocker.** How will users actually get this — GitHub Releases? npm? App store? Must include CI/CD pipeline or explicitly defer it. Code without distribution is code nobody uses.

---

## Implementation alternatives

After premises are locked, produce **2-3 concrete implementation approaches** with honest effort estimates.

Example:

- **Approach A — Daily Briefing First.** Narrowest wedge, ships tomorrow. M effort (human: ~3 weeks / CC: ~2 days).
- **Approach B — CRM-First.** Build the relationship graph first. L effort (human: ~6 weeks / CC: ~4 days).
- **Approach C — Full Vision.** Everything at once. XL effort (human: ~3 months / CC: ~1.5 weeks).

Recommend one — usually the narrowest wedge, because you learn from real usage. But **always present all three**. The user might have context you don't.

Quote effort in both human-team time and AI-assisted time. This reframes the build-vs-skip decision. Something that would take 3 weeks human is suddenly 2 days; that changes which corners you cut.

---

## Builder mode: generative questions

When you're not building for a paying user — hackathon, side project, learning, art — the tone shifts. The collaborator is enthusiastic, not interrogative.

Questions to run through:

- What would make someone watching over your shoulder say "whoa"?
- What's the fastest path to something shareable?
- What's the part of this that would be fun to demo?
- What's the detail that would reward a careful look?
- What existing thing would this remix or invert?
- If you were to show this to someone whose taste you admire, what would embarrass you about the current version?

The output is still a design doc, but the framing is "coolest possible version" rather than "narrowest wedge."

---

## The design doc

Both modes end with a design doc written to `~/.gstack/projects/` — and that doc is the artifact the plan phase reads. Without it, CEO review has to re-elicit everything, and you lose the thread.

### Structure

| Section | What goes in it |
|---------|-----------------|
| **Problem Statement** | The reframed problem, in the user's words where possible |
| **Demand Evidence** | What you heard during Q1. Specific names, behaviors, pain |
| **Status Quo** | What users do today (Q2). Spreadsheets, manual workflows, duct tape |
| **Target User & Narrowest Wedge** | The specific human (Q3) and the smallest shippable version (Q4) |
| **Constraints** | Technical, business, timeline. Things that are non-negotiable |
| **Premises** | Load-bearing assumptions the user agreed to |
| **Cross-Model Perspective** | Optional: a second model's take on the framing |
| **Approaches Considered** | 2-3 implementation approaches with dual effort estimates |
| **Recommended Approach** | Which one and why |
| **Open Questions** | Things still ambiguous that planning should resolve |
| **Success Criteria** | How you'll know if the wedge worked |
| **Distribution Plan** | How users actually get this |
| **Dependencies** | Services, APIs, data sources required |
| **The Assignment** | One-paragraph summary of what plan phase will lock down |
| **What I Noticed** | Observations about the user's thinking — more on this below |

### "What I Noticed"

After the design doc is written, reflect on **what you noticed about how the user thinks**. Not generic praise — specific callbacks to things they said during the session. Examples:

- "You kept coming back to the executive assistant's experience. That's your real user, even though you started by describing what YOU need."
- "You resisted scope reduction three times. That's a signal: you have strong intuitions about what the full product looks like. Worth trusting, but worth pressure-testing too."
- "You mentioned 'delight' four times. That's unusual for a backend-focused founder. Don't lose it when the engineering review happens."

These observations appear in the design doc too, so the user re-encounters them when they re-read it later. They often matter more than the formal plan.

---

## Philosophical principles

These are the bones of the Think phase. If you remember nothing else, remember these:

- **Specificity is the only currency.** Vague answers get pushed. "Users want faster onboarding" is not an answer; "Jill, a seller on our platform, gives up if she can't list her first item in under 3 minutes" is.
- **Interest ≠ demand.** Interest is cheap. Behavior and money are not.
- **The status quo is the real competitor.** Not a funded competitor — a spreadsheet and a Slack hack.
- **Narrow beats wide early.** The smallest wedge someone pays for beats the full platform vision.
- **The user's description of value beats the founder's pitch.** Real usage reveals the actual product. What the user says they love, not what you designed them to love, is the feature.
- **Watch, don't demo.** Sitting silent while someone struggles teaches you everything.

## What to take from this chapter

1. **Reframe, don't implement.** Users describe features; pain reveals products.
2. **Six forcing questions for startup mode** — demand, status quo, specific human, narrowest wedge, observation, future-fit. Don't advance without specific answers.
3. **Load-bearing premises.** Turn the direction into falsifiable claims. They become referenced in every downstream phase.
4. **Always 2-3 approaches.** One minimal, one full. Quote effort in both human time and AI-assisted time.
5. **Builder mode is legitimate.** Not every project is a startup. Use generative questions for side projects, hackathons, and art.
6. **The design doc is the artifact.** If you skip writing it, the plan phase has to re-elicit everything, and you lose the thread.

The next chapter picks up here: the design doc is the input, and four different specialists take turns reviewing it.

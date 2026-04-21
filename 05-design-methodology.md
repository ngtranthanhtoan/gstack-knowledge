# 05 — Design End-to-End

Design is where AI fails hardest by default. Engineering problems have tests that can say "passed." Design problems have no unit test for "looks intentional." That gap is why the web is full of sites that look the same — purple gradient, three-column feature grid, rounded-corner cards with drop shadows — and why specifying design in the plan phase is the single highest-leverage move gstack recommends.

This chapter covers the full design pipeline: building a design system from nothing (**consultation**), exploring options visually (**shotgun**), turning an approved design into HTML that actually works (**design-html**), and auditing live sites for the patterns that betray AI authorship (**design-review**). It closes with the AI Slop Canon: a working inventory of what to stop doing.

## Why design needs its own pipeline

Engineering and design both benefit from AI, but they benefit differently. Engineering tasks have objective correctness: the test passes or it doesn't, the query plan hits the index or it doesn't. Design tasks have taste correctness: two implementations can both work and one still feels like a ghost. You can only catch the ghost by looking.

gstack's design pipeline reflects this: every step has a visual checkpoint, and no approval happens without comparison against alternatives.

---

## Part 1 — Building a design system from scratch (`/design-consultation`)

When you're starting from nothing — no fonts chosen, no palette, no spacing scale — sit down with a design partner and build the whole visual identity together. Not a form; a conversation.

### The consultation arc

1. **Product context.** What is this? Who's it for? What space/industry?
2. **Aesthetic direction.** Pick a feel — brutally minimal, maximalist, retro-futuristic, luxury, playful, editorial, brutalist, art deco, organic, industrial/utilitarian. Each pairs with specific decoration levels and layout approaches.
3. **Competitive research.** Optionally: the agent browses competitors, takes screenshots, analyzes their fonts/colors/spacing. You see the landscape before you make choices.
4. **Typography.** 3+ fonts, each with a specific role (display, body, UI, data, code). Specific weights. Specific loading strategy.
5. **Color.** Primary, neutrals, semantic. Dark mode and light mode both resolved.
6. **Spacing and layout.** Base unit, density, grid, max content width, border radius by element role.
7. **Motion.** Minimal-functional, expressive, playful. Easing curves and durations named.
8. **Realistic preview.** The agent renders a dashboard, a form, a marketing page — real product surfaces in your system, in your voice, in light and dark mode.
9. **`DESIGN.md`** — the source of truth written to the repo. `CLAUDE.md` updated so every future session respects it.

### The "safe choices vs risks" framing

Every dev-tool dashboard looks the same — clean sans-serif, muted grays, a blue accent. They're all coherent. They're all forgettable.

The difference between a product that looks "nice" and one that people **recognize** is the deliberate creative risks: the unexpected serif for headings, the bold accent nobody else in your category uses, the tighter spacing that makes your data feel authoritative instead of airy.

A good consultation **names both**. Example output:

> **SAFE CHOICES** (match category expectations):
> - Geist for body — your users already know this font from Vercel.
> - Slate neutrals — warm grays feel right for data-heavy UIs.
> - Grid-disciplined layout — dashboards need predictable alignment.
>
> **RISKS** (where you'd stand out):
> - Instrument Serif for display headings — nobody in dev tools uses a serif. Creates instant hierarchy contrast and signals "we care about craft."
> - Single teal accent instead of blue — everyone else in this space uses blue. Teal is close enough to feel trustworthy, different enough to be recognizable.
> - 4px compact density — tighter than Linear, says "we respect your screen real estate."
>
> The safe choices keep you literate. The risks are where your product gets its own face.

The user picks which risks to take. The system stays coherent either way.

### The aesthetic direction menu

A useful starting taxonomy:

| Direction | Feel | Typical pairing |
|-----------|------|-----------------|
| **Brutally Minimal** | Restrained, monospace personality, no decoration | Geist + JetBrains Mono, black/white + 1 accent |
| **Maximalist Chaos** | Bold, layered, overlapping | Display serif + sans, saturated palette |
| **Retro-Futuristic** | 80s-90s digital nostalgia | Custom display, neon + dark, CRT textures |
| **Luxury / Refined** | Generous whitespace, elegant typography | Serif display, muted warm neutrals |
| **Playful / Toy-like** | Cartoonish warmth | Rounded sans, bright primaries |
| **Editorial / Magazine** | Text-first, hierarchy-driven | Serif display + sans body, grid-heavy |
| **Brutalist / Raw** | Exposed, intentional harshness | Monospace, high-contrast, utilitarian |
| **Art Deco** | Symmetry, gold accents, geometric | Custom display, cream + gold + navy |
| **Organic / Natural** | Muted earth tones, hand-feel | Humanist sans, warm neutrals, grain texture |
| **Industrial / Utilitarian** | Function-first, monospace personality, data density | Geometric sans + mono, zinc + amber |

Every direction is coherent. The wrong one for your product is still a worse choice than a generic one.

### Font blacklist vs. gold standards

**Never.** Papyrus. Comic Sans. Lobster. Impact. Brush Script. Anything that reads as a meme.

**Also: overused.** Inter, Roboto, Arial, Helvetica, Poppins, Montserrat. Technically fine. Make your product indistinguishable from every other SaaS.

**Display gold standards.** Satoshi, General Sans, Instrument Serif, Fraunces, Cabinet Grotesk.

**Body gold standards.** Geist, DM Sans, Source Sans 3, Plus Jakarta Sans.

**Data / code.** JetBrains Mono. Supports tabular-nums. Accept no substitutes.

### An example artifact

From gstack's own `DESIGN.md`:

- **Direction:** Industrial/Utilitarian — function-first, data-dense, monospace as personality font.
- **Mood:** Serious tool built by someone who cares about craft. Warm, not cold. The CLI heritage IS the brand.
- **Display:** Satoshi Black/Bold — geometric with warmth, distinctive letterforms.
- **Body:** DM Sans Regular/Medium/Semibold.
- **Data/Tables:** JetBrains Mono. Should be prominent, not hidden in code blocks.
- **Primary:** amber-500 (dark), amber-600 (light) — warm, energetic, reads as "terminal cursor."
- **Neutrals:** Cool zinc grays.
- **Spacing base:** 4px. Comfortable density — not Bloomberg, not marketing site.
- **Grain texture:** subtle SVG turbulence overlay (0.03 dark / 0.02 light) for materiality.

Every decision has a **Decisions Log** appended with date and rationale. When you ask six months later "why amber-600 in light mode?" the answer is there: *"amber-500 too bright/washed against white; amber-700 too brown. amber-600 is the sweet spot."*

### Principles

- **Coherence beats individual optimization.** Every choice reinforces every other choice.
- **Explain reasoning for every recommendation.** "Because it's clean" is not a reason.
- **Research the landscape before proposing.** Know what exists so you know which conventions to break.
- **Coherence is table stakes. Creative risk is the product.**

---

## Part 2 — Visual exploration (`/design-shotgun`)

You have a feature or a page and you're not sure what it should look like. Describing it gets you one answer. One answer is one perspective; design is a taste game. You need to see options.

### The exploration loop

1. **You describe what you want** (or point at an existing page).
2. **The skill reads `DESIGN.md`** for brand constraints if one exists.
3. **It generates 3-6 distinct variants** as PNGs using an image model.
4. **A comparison board opens in your browser** — all variants side-by-side.
5. **You click "Approve"** on one, or leave feedback ("more whitespace", "bolder headline", "lose the gradient") for another round.
6. **The approved variant saves** to `~/.gstack/projects/$SLUG/designs/` with an `approved.json`.

### Distinct, not varied

"Generate 3 variants" is tempting to read as "three versions of the same idea." It isn't. Variants should be **distinct concepts**: different layout strategies, different emotional tones, different structural ideas. If all three variants share the same layout with minor color shifts, the generator didn't diverge enough.

Example for a developer-tool landing hero:

- **Variant A** — Bold typography, dark background, code snippet as hero.
- **Variant B** — Split layout, product screenshot left, copy right.
- **Variant C** — Minimal, centered headline, single gradient accent.

These are not "variations." They're different products.

### Taste memory

The skill remembers your preferences across sessions. If you consistently approve minimal over busy, later generations bias toward minimal. If you always pick dark mode hero shots, later rounds default to dark. The memory is **emergent from approvals**, not a setting you configure.

This matters because design is bad at articulating itself. You can't always say *why* you approved Variant A over B. The memory captures the pattern even when you can't.

### AI slop detection in variants

If a generated variant looks like generic stock-photo hero sections, centered-everything layouts, or decorative blobs not in your brief, flag it. Regenerate. Variants should feel intentional, not default-generator output.

---

## Part 3 — Design to code (`/design-html`)

Every AI code generation tool produces static CSS. Hardcoded heights. Text that overflows on resize. Breakpoints that snap instead of flowing. The output looks right at exactly one viewport size and breaks at every other.

This is the single biggest reason AI-generated landing pages feel "off" — they look fine in the screenshot and broken on your actual device.

### The Pretext insight

`/design-html` fixes this by using [Pretext](https://github.com/chenglou/pretext), a 15KB library that computes text layout *without* DOM measurement. Text reflows. Heights adjust to content. Cards self-size. Chat bubbles shrinkwrap. All sub-millisecond, all dynamic.

Without computed layout, "responsive design" is a promise. With it, it's a fact.

### The four-step wiring pattern (simplified)

1. **PREPARE** — one-time text measurement after fonts load.
2. **LAYOUT** — cheap, called on every resize (sub-millisecond).
3. **RESIZE-AWARE** — `ResizeObserver` re-layouts dynamically.
4. **CONTENTEDITABLE** — `MutationObserver` re-prepares when text changes.

### Smart API routing per design type

Not every page needs the full Pretext engine. Pick the right primitive:

| Design type | Pretext pattern |
|-------------|-----------------|
| Simple layouts (landing, marketing) | `prepare()` + `layout()` — resize-aware heights |
| Card/grid layouts (dashboards, listings) | `prepare()` + `layout()` — self-sizing cards |
| Chat UIs | `prepareWithSegments()` + `walkLineRanges()` — tight-fit bubbles |
| Editorial (text flowing around images) | `prepareWithSegments()` + `layoutNextLine()` |
| Complex editorial | Full engine with `layoutWithLines()` — manual line rendering |

### Input flexibility

`/design-html` works with whatever you have:

- An approved mockup from `/design-shotgun`.
- A CEO plan from `/plan-ceo-review`.
- Design review context from `/plan-design-review`.
- A PNG you provide.
- Just a description.

It detects what's present and asks how you want to proceed.

### The refinement loop

1. Reads the approved mockup (`approved.json`).
2. Uses vision to extract implementation spec: colors, typography, layout, spacing.
3. Generates self-contained HTML with Pretext inlined (15KB, zero network dependencies).
4. Spins up a live-reload server — changes visible instantly.
5. Screenshots at 3 viewports (375px, 768px, 1440px) to verify layout.
6. Asks: what needs to change?
7. **Surgical edits** via the Edit tool, not full regeneration. (The user may have made manual edits via `contenteditable`.)
8. Repeat until "done."

### Framework detection

If the project uses React, Svelte, or Vue (detected from `package.json`), the skill offers to generate a framework component instead of vanilla HTML. Framework output uses `npm install @chenglou/pretext` instead of inline vendoring.

---

## Part 4 — Live-site audit (`/design-review`)

`/plan-design-review` reviews the plan before implementation. `/design-review` audits and fixes the live site after.

### The method

1. **80-item visual audit** on the live site — typography, spacing, hierarchy, consistency, interactive states, accessibility, visual performance.
2. **Two rated scores**:
   - **Design Score (0-100)** — overall quality.
   - **AI Slop Score (0-100)** — how much of the site feels generic/AI-generated.
3. **Findings classified** by severity: high, medium, polish.
4. **Fix loop.** For each finding: locate the source file, make the minimal CSS/styling change, commit as `style(design): FINDING-NNN — short description`, re-navigate to verify, take before/after screenshots.
5. **One commit per fix.** Fully bisectable.
6. **Risk budget.** CSS-only changes get a free pass (inherently safe and reversible). Changes to component JSX/TSX files count against the risk budget. Hard cap: 30 fixes. If risk exceeds 20%, stop and ask.

### Example output

```
Design Score: C  |  AI Slop Score: D
12 findings (4 high, 5 medium, 3 polish)

Fixing 9 design issues…

style(design): FINDING-001 — replace 3-column icon grid with asymmetric layout
style(design): FINDING-002 — add heading scale 48/32/24/18/16
style(design): FINDING-003 — remove gradient hero, use bold typography
style(design): FINDING-004 — add second font for headings
style(design): FINDING-005 — vary border-radius by element role
style(design): FINDING-006 — left-align body text, reserve center for headings
style(design): FINDING-007 — add hover/focus states to all interactive elements
style(design): FINDING-008 — add prefers-reduced-motion media query
style(design): FINDING-009 — set max content width to 680px for body text

Final audit:
Design Score: C → B+   |   AI Slop Score: D → A
9 fixes applied (8 verified, 1 best-effort). 3 deferred.
```

The AI Slop score jumped from D to A because the three most recognizable patterns (gradient hero, 3-column grid, uniform radius) are gone.

### Verification at 3 viewports

Before/after screenshot pairs at 375px (mobile), 768px (tablet), 1440px (desktop). Catches the responsive breakage that "looks fine in the editor" hides.

---

## Part 5 — The AI Slop Canon

These are the patterns that mark a design as AI-generated. They appear across multiple gstack skills as "never do this."

### The visual canon

1. **Gradient everything.** Especially purple/blue as the default accent. Purple gradients are the single biggest tell.
2. **Generic feature grids.** 3-column card layouts with icons in colored circles.
3. **Centered uniformity.** Everything centered, same spacing, no hierarchy.
4. **Decorative bloat.** Blobs, waves, patterns not in the approved design.
5. **Stock imagery.** Hero sections that look like Getty Images.
6. **Generic CTAs.** "Get Started." "Learn More." "Try Now." No specificity.
7. **Cookie-cutter cards.** Rounded corners + drop shadow + padding — the default card template.
8. **Emoji as visual elements.** Using 🚀 or 💡 to carry the visual weight.
9. **Generic testimonial sections.** Grid of circular photos with two-sentence quotes.
10. **Left-text-right-image heroes.** The most common AI hero layout because it's the easiest to generate.

### The copy canon

11. **"Built for X."** "Designed for Y." Marketing copy that says nothing about the product.
12. **"The future of [thing]."** Vague grandiose positioning.
13. **"Simple, fast, reliable."** The three-word adjective pile that means nothing.

### The typography canon

14. **Overused fonts.** Inter, Roboto, Arial, Helvetica, Poppins, Montserrat.
15. **Banned fonts.** Papyrus, Comic Sans, Lobster, Impact, Brush Script.

### The behavior canon

16. **Responsive brittleness.** Layouts that break on specific viewports.
17. **Missing states.** Happy path only — no error, loading, empty states.
18. **Inconsistent components.** The same button looks different on three pages.
19. **Uncontrolled spacing.** Random padding instead of a consistent scale.
20. **Hierarchy collapse.** Not clear what to read first, second, third.

### The anti-pattern: all ten at once

A site that has every one of these is a site generated by an AI that was told *"make a landing page for a developer tool"* with no further specification. You can spot them at thirty paces. They all blur together in memory because they are, in a real sense, the same page.

The goal of design review, plan-design-review, and the consultation pipeline is to catch these before they ship — because a single one gets through and your product reads as generic; five get through and it reads as AI.

---

## What to take from this chapter

1. **Design has its own pipeline** because it needs visual checkpoints at every step.
2. **Consultation builds the system.** Name safe choices AND creative risks. Coherence beats individual optimization.
3. **Shotgun explores visually.** Distinct concepts, not variations. Taste memory emerges from approvals.
4. **Pretext-based HTML avoids static CSS.** Text reflows, heights adjust, layouts dynamic — across viewports.
5. **Live-site audits score both design and AI slop.** Fix in atomic commits with before/after screenshots.
6. **The AI Slop Canon is the single most useful artifact.** Memorize it. Every pattern it lists is a default mode you have to actively avoid.
7. **Design is intentional or accidental.** Every pixel either builds or erodes trust.

The next three chapters shift from "build the right thing well" to "catch what's still broken" — code review, investigation, and QA.

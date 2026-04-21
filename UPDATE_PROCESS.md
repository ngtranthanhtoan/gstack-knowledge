# Update Process Memo

This directory is a **derivative knowledge book** extracted from the gstack repo at `../gstack/`. gstack itself is upstream — we periodically pull new code from its origin. When that happens, the books in this directory need to be updated to match.

This memo explains how that update cycle works.

## The checkpoint

[checkpoint.txt](checkpoint.txt) records the timestamp of the last time the gstack repo was synced from its origin. Example:

```
Lastest pull: 2026-04-18 17:13:51 +07
```

This timestamp is the **baseline** for all the knowledge currently captured in chapters 01-12. Everything in those chapters reflects the state of `../gstack/` **at that moment**.

## The update cycle

When the gstack repo is pulled again from origin, the workflow is:

1. **Pull upstream.** `cd ../gstack && git pull` (or equivalent).
2. **Note the new timestamp.** The new checkpoint value is "now."
3. **Ask the AI to update this book.** The AI reads the diff between the old checkpoint and the new HEAD, and decides what knowledge to add, modify, or remove in this directory.
4. **Update the checkpoint.** Once the book is in sync, `checkpoint.txt` is overwritten with the new timestamp. That becomes the new baseline for next time.

## What the AI should do on an update

Given the old checkpoint and the current state of `../gstack/`, the AI should:

### 1. Compute the diff

```bash
cd ../gstack
git log --since="<old checkpoint>" --name-status
git diff <commit-at-old-checkpoint>..HEAD --stat
```

Focus on files that hold **knowledge**, not plumbing:

- `ETHOS.md` — philosophy changes.
- `ARCHITECTURE.md` — architectural principles.
- `DESIGN.md` — design system details.
- `README.md` — high-level framing.
- `docs/skills.md` — skill deep-dives (the most human-readable source).
- `docs/ON_THE_LOC_CONTROVERSY.md` and other `docs/*.md` — standalone essays.
- `docs/designs/*.md` — new methodology designs.
- `*/SKILL.md.tmpl` — skill methodology (where most craft knowledge lives).

Ignore changes to `*/dist/`, `package.json`, `bun.lock`, `test/`, `scripts/`, and other implementation-only files unless the diff signals a new methodology being formalized.

### 2. Classify the changes

For each changed source file, the AI should classify:

| Type | What to do |
|------|-----------|
| **New skill added** | If the new skill introduces a novel methodology, add a section to the most relevant existing chapter. If it's a major new phase, consider a new chapter. |
| **Existing skill methodology changed** | Update the corresponding section in the relevant chapter. Preserve the voice and structure of the book. |
| **Philosophy / ETHOS changes** | Update [01-the-builder-ethos.md](01-the-builder-ethos.md). |
| **Architecture internals** | Update the architecture notes in [12-safety-and-parallel-work.md](12-safety-and-parallel-work.md) (Part 3). |
| **New rubric, checklist, or rule** | Preserve it verbatim. These are working artifacts, not summaries. |
| **Skill deprecated / removed** | Remove or mark as deprecated in the relevant chapter. Don't silently delete — note the shift. |
| **Pure plumbing** (tests, CI, build scripts) | Skip — not knowledge. |

### 3. Respect the book's voice and structure

- The book is human-readable prose, not a skill-instructions dump. Translate *"the agent should…"* into *"when you do this…"*.
- Preserve gstack's voice where it's load-bearing (direct, opinionated). Garry's directness is part of the signal.
- Keep chapter boundaries stable unless there's a strong reason to reorganize. Readers of v1 should be able to find things in v2.
- If a chapter grows past ~400 lines during an update, consider splitting it.
- Never remove rubrics, checklists, or specific rules without replacement. These are the highest-leverage parts of the book.

### 4. Write the update

Apply edits. Prefer `Edit` over `Write` so diffs stay small. For new sections, find the nearest existing section and insert alongside it — don't append to the end of the file.

### 5. Update the checkpoint

Once the book reflects the new upstream state, overwrite `checkpoint.txt` with the new timestamp:

```
Lastest pull: <YYYY-MM-DD HH:MM:SS TZ>
```

That becomes the baseline for the next update cycle.

### 6. Optional: keep a changelog

For significant updates, consider adding a line to a `CHANGELOG.md` in this directory (not yet present — create it on the first substantive update) describing what shifted in the extracted knowledge. Like all changelogs: what a reader can now learn that they couldn't before, not implementation detail.

## What NOT to do on an update

- **Don't regenerate the whole book.** Incremental edits preserve the reader's ability to re-read familiar chapters without starting over.
- **Don't chase every small change.** Typo fixes, build script tweaks, and internal refactors in `../gstack/` don't belong in this book.
- **Don't delete knowledge just because the upstream skill name changed.** A renamed `/plan-ceo-review` → `/ceo-plan` is a text update, not a methodology change. Keep the knowledge; update the reference.
- **Don't merge chapters just to make the book shorter.** The 12-chapter structure matches the sprint phases. Conflating chapters blurs the mental model.
- **Don't invent knowledge the source doesn't contain.** If a gstack change is cryptic, ask the user rather than fabricating methodology to explain it.

## Quick reference for future AI sessions

If you're reading this because the user asked you to update the book:

1. Read `checkpoint.txt` to get the last-sync timestamp.
2. `cd ../gstack && git log --since="<timestamp>"` to see what changed.
3. Classify the changes using the table above.
4. Make incremental edits to the affected chapters.
5. Update `checkpoint.txt` with the new timestamp.
6. Report what changed to the user in one paragraph.

The goal is always the same: **the book should read as if it was written today, about the current state of gstack** — and a reader should be able to trust every rubric, rule, and principle as still current.

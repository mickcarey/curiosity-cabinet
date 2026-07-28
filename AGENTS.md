# AGENTS.md

## Project Type: Curiosity Cabinet 🎭

This is a Markdown-first notes and documentation project: a collection of thoughts, ideas, research trails, biographies, article notes, website analyses, and useful curiosities.

This is not an application repo. No app to compile. No service to deploy. No tiny server pretending it is "just here for the snacks."

## Key Rules

### 📝 File Creation

- Only create or edit Markdown (`.md`) files.
- Do not add code files, config files, package files, generated assets, scripts, or build tooling.
- Keep changes focused on writing, structure, links, sources, and knowledge organization.
- Preserve existing user-authored content unless the user explicitly asks for removal.

### 🎨 Writing Style

- Keep notes fun, playful, and curious. This is a curiosity cabinet, not a corporate manual wearing sensible shoes.
- Use personality, whimsy, emojis, metaphors, and creative language where they help.
- Be conversational rather than sterile or professionally beige.
- Explain clearly first, then make the idea memorable with a good analogy, vivid image, or sharp little turn of phrase.
- Use emojis as visual colour in titles, section headings, article lists, and callouts. They should help the page breathe and break up dense text.
- Let the topic set the volume. Light or eccentric subjects can be bouncy and emoji-rich; serious subjects can still be witty, but the humour should respect the weight of the material.
- If the writing starts trying too hard to be funny, simplify. The joke should serve the idea, not leap in front of it wearing tap shoes.

Good target: "smart friend explaining the cool part, with enough humour and colour that the facts actually stick."

Bad target: "technical documentation, academic fog, or a comedy routine stapled to a biography."

### 💭 Content Approach

- Embrace curiosity and wonder.
- Make connections between ideas when they are genuinely useful.
- Include interesting tangents, quirks, contradictions, and rabbit holes.
- Format notes in an engaging, readable way with headings, short sections, bullets, quotes, and links where useful.
- Prefer vivid clarity over exhaustive-but-dull coverage.
- Include brief personal reactions or "why this matters" moments when they sharpen the note.

Think "museum of interesting thoughts" rather than "technical documentation."

## 📂 Content Organization

### 👤 Biographies

When asked to add a new biography, profile, or person:

- Create the file in `people/`.
- Name it with lowercase words separated by hyphens, such as `ada-lovelace.md`.
- Include what makes the person fascinating, not just a timeline.
- Add interesting facts, quirks, contradictions, signature ideas, notable work, and why they matter.
- Where helpful, include rabbit holes or things to look up next.

Suggested shape:

```markdown
# Person Name

## Why They Matter

## The Interesting Bits

## Signature Ideas

## Strange But True

## Follow-Up Rabbit Holes
```

### 🌐 Website Analyses

When asked to analyse a website or add a website analysis:

- Create the file in `websites/`.
- Name it after the domain or site name, such as `wonjyou-studio.md`.
- Include design observations, unique features, content strategy, interaction details, engineering clues, and what makes the site notable.
- Separate observation from inference. If something is a guess, say so.

Suggested shape:

```markdown
# Website / Domain

## First Impression

## Design Notes

## Interaction Notes

## Content Notes

## Engineering Clues

## Why It Is Interesting

## Follow-Up Rabbit Holes
```

### 📚 Article Collection

When asked to save an article or add an article link:

- Update `articles/index.md`.
- Add it to a relevant file in `articles/by-topic/` when it fits an existing or useful theme.
- Add it to the appropriate monthly log in `articles/reading-log/`, such as `2026-07.md`.
- Create missing Markdown files or directories as needed.
- Use the current local date unless the user gives a specific date.

Article entries should use this format:

```markdown
### [Article Title](url)
*Source • Date read*

Quick note about why this caught attention, the key insight, or what it connects to.
```

## 🔎 Research And Sources

- Prefer reliable sources when facts matter.
- Include source links for specific, current, disputed, or easy-to-misremember claims.
- If browsing or external research is used, cite the source in the note.
- Be clear about uncertainty.
- Do not overquote. Summarize in your own words unless a short quote is especially useful.

## 🌿 Git Workflow

Before starting a change:

- Check the current branch and worktree state.
- If there is existing user work, do not overwrite or revert it.
- Start new work from `main` unless the user asks otherwise.
- Make sure `main` is up to date with the remote before branching.
- Never commit directly to `main`.
- Never push directly to `origin/main`.
- Use a separate branch and open a PR for review.
- Use squash merges for completed PRs.

```bash
git switch main
git pull --ff-only
git switch -c short-descriptive-branch-name
```

## Remember

This is a space for collecting and organizing interesting ideas, not for writing code. Every file should spark curiosity or capture a fascinating thought.

---
name: html
description: Produce a single self-contained .html file instead of a markdown wall when the answer has shape — side-by-side comparisons, a timeline, a diagram, a sortable list, or a deck. This skill should be used when the user invokes /html or any of its patterns (options, plan, pr, codemap, tokens, motion, diagram, deck, explainer, status, editor), and when a request implies a structured artifact — phrases like "show me my options for X", "write the plan", "do a post-mortem", "make a deck from these notes", "let me triage these", "render the design system", "explain how X works", "diagram the flow". Prefer this skill even when the user says "write me a doc" or "summarize" — HTML is often the better surface, and the user can always ask for markdown if they prefer.
---

# html

You are producing **a single self-contained `.html` file**, not a markdown document. The reader will open it directly in a browser. No build step, no framework, no install. Treat the file as the deliverable.

Based on Thariq Shihipar's [*The unreasonable effectiveness of HTML*](https://thariqs.github.io/html-effectiveness/) — the observation that markdown flattens information that is inherently spatial, comparative, or interactive, and a small HTML file fixes this.

## What this skill is for (read this first)

**Agent-generated artifacts for a human reader.** Throwaway or reference, not production.

Concretely, the things this skill produces are:

- **Working documents** the reader will read once and act on (a plan, an options comparison, a PR writeup, a status report).
- **Reference pages** the reader will return to (a design-tokens page, a codemap, an explainer).
- **Throwaway tools** the reader uses for a few minutes to do a piece of work, then exports out of (a triage board, a feature-flag toggler, a motion tuner).

What this skill is **not** for:

- **Production websites or apps** shipped to end users. The conventions here (single file, no build step, no framework, no server, vanilla JS, in-memory state) exist for fast disposability, not for production quality. If the user is building something they ship to customers, this is the wrong tool — use a real frontend skill.
- **Long-form prose** with no shape (an email, a paragraph of analysis, a conversational reply). HTML doesn't earn its place when the content is just sentences. Use markdown.
- **Things that need persistence, multi-user state, or a backend.** State here lives in memory and dies on refresh. The export button is how work survives.

If you're not sure which side a request falls on, ask yourself: would a human be happy reading this once in a browser tab and then closing it? If yes, this skill applies. If they need to bookmark it, share it with twenty colleagues, and have it still work in six months — that's a different skill.

## When this skill applies (signals)

If the user invoked `/html` or a pattern slash command directly, just produce HTML. They've already chosen.

Otherwise, the strong signals are:

- **Comparison** — "show me my options", "compare X vs Y", "trade-offs of A, B, C"
- **Code review or PR** — annotate a diff, write the PR description, map a codebase
- **Design** — tokens, component variants, contact sheet
- **Motion** — animation, transition, click-through where feel matters
- **Diagram** — flowchart, architecture, data flow, illustrative figures
- **Deck** — a short presentation you'd arrow-key through
- **Explainer** — concept or feature, with collapsibles and tabs
- **Recurring report** — weekly status, incident timeline, post-mortem
- **Editor** — a throwaway UI for the specific thing the user is wrestling with
- **Table** — rows × columns the reader needs to sort, filter, or scan

If the request is plain prose without shape, use markdown. HTML is not a universal hammer; it earns its place when the content has structure.

## What every output looks like

Every artifact this skill produces obeys these conventions. Patterns assume you've followed them.

1. **Single file.** One `.html`. No external CSS, no external JS, no build step. Inline everything.
2. **No network at render time.** No CDN, no Google Fonts, no remote images. The artifact must render offline.
3. **Locked look — Claude editorial system.** Start from `templates/base.html`. Do not redesign tokens per artifact. The look (cream canvas `#faf9f5`, coral accent `#cc785c`, dark navy `#181715`, EB Garamond serif display at weight 400 with negative tracking, Inter / system sans body, JetBrains Mono code) is fixed across every artifact this skill produces so the reader recognizes the genre on sight. If you need to tune a token, change `base.html`, not the per-artifact `<style>`.
4. **Mobile-readable.** Viewport meta is in the template. Multi-column layouts collapse below ~720px.
5. **Semantic HTML.** `<section>`, `<article>`, `<details>/<summary>`, `<figure>`, `<table>`. Headings nest properly.
6. **Information has shape.** Don't pour markdown into a `<div>`. Side-by-side things go in a grid. Sequences go on a timeline. Risks go in a table. If the content has no shape, you're producing the wrong artifact — use markdown.
7. **A "Prompt:" block near the top.** Small italic callout with the user's request, near-verbatim. Makes the artifact legible later and makes it forkable. Already styled in the template as `.prompt`.
8. **Inline SVG over images.** The agent has a real pen — use it. Diagrams are `<svg>`, not `<img>`.
9. **Vanilla JS only, sparingly.** One `<script>` tag at the bottom. State in a single in-memory object. No React, no Vue, no jQuery, no framework. No `localStorage` unless explicitly requested.
10. **Close the loop: export back to text.** Any artifact the user can *do something to* (toggle, reorder, tune, edit) MUST end with a button that emits the result as something pasteable — markdown, JSON, a diff, a prompt. The artifact is throwaway; the export is the durable output. This is the single most important convention.
11. **Export-first design.** Decide what text the user will copy out *before* designing the UI. The data model serves that export; the UI mutates the data model; the data model must always be derivable into the export string. Applies to any interactive pattern — `editor`, `motion`, `tokens`, `table`, and interactive variants of `plan` or `pr`. Detailed in `patterns/editor.md`.
12. **Live tuning uses primitives from the gallery.** When the user asks to tune values live (sliders, toggles, drag-and-drop, sort, filter, swatch pickers), lift the primitive from `gallery/interactions.html` rather than writing one from scratch. Keeps interaction style consistent across artifacts.

## How to start

In this order, every time:

1. **Identify the pattern(s).** Skim the table below. Pick the closest one. If a request spans two patterns (planning + comparison, explainer + diagram), pick a primary and borrow structure from the secondary — don't refuse to combine.
2. **Read the relevant pattern playbook(s).** `patterns/<name>.md`, ~70–150 lines each, opinionated and prescriptive. The playbook owns the *shape* of the artifact: sections, order, what the export emits.
3. **Read the shared template.** `templates/base.html` — boilerplate, CSS variables, utility classes. Start from this rather than rewriting it. **Do not redefine the tokens.**
4. **Decide the export format before the UI** (if interactive). What text would the user paste somewhere after they're done? Markdown? A diff? A JSON blob? A re-issuable prompt? The data model serves that export.
5. **Write the file.** One pass: head → header (title + prompt block) → main content → export button + script.

### Reference files (don't copy structure from these)

The `gallery/` folder is a **look + interaction reference**, not a shape reference. Each file shows primitives in isolation; structure for the artifact you're producing comes from the pattern playbook, not from gallery files.

- **`gallery/look.html`** — color swatches, type ramp, callout / chip / card / glance / timeline variants. After producing an artifact, mentally compare its surfaces, type, and chips to this file. If they don't match, your tokens drifted — fix them.
- **`gallery/interactions.html`** — sliders, toggles, switches, drag-and-drop columns, sortable lists, filter bars, segmented controls, swatch pickers, number steppers. Pull the primitive you need when the user wants to tune values live.
- **`gallery/navigation.html`** — tabs, arrow-key deck, sticky side-nav, anchor row, collapsible details, sticky export bar. Pull from here when the artifact needs to move the reader around.

## Where the file goes

- If a working directory or output path is implied, write there.
- Otherwise: current working directory, kebab-case filename from the topic. `debounced-search-options.html`, `cycle-14-triage.html`, `comment-threads-plan.html`.
- After writing, tell the user the path and suggest they open it.

## Patterns

Each lives at `patterns/<name>.md`. Read the one you need.

Universal patterns (work for any domain — software, marketing, research, ops, life admin):

| Pattern | When to use |
|---|---|
| `options` | "Show me 2–4 ways to do X with trade-offs" — any decision before commitment |
| `plan` | A milestoned plan with a timeline, mockups or sketches, and a risk table |
| `diagram` | Hand-rolled SVG diagrams — flowcharts, architecture, illustrative figures |
| `deck` | A short presentation, arrow-key navigable, single file |
| `explainer` | Layered explanation — TL;DR, collapsibles, tabs, glossary |
| `status` | Recurring report — weekly status, project update, post-mortem, retro |
| `editor` | Throwaway UI for editing the specific thing at hand, with export-back-to-text |
| `table` | Sortable, filterable tabular data — incidents, customers, line-items, anything with rows and columns |

Software-specific patterns (assume the user is working with code; descriptions reflect this):

| Pattern | When to use |
|---|---|
| `pr` | Code-review writeup — annotated diff OR author's PR description |
| `codemap` | Onboarding to an unfamiliar codebase — boxes, arrows, hot path, entry points |
| `tokens` | Render a design system — live swatches, type ramp, component contact sheet |
| `motion` | UI animation or click-through prototype where feel can't be described in prose |

## Combining patterns

Real requests cross pattern boundaries. That's normal — combine, don't refuse.

Common combinations:

- **plan + options.** "Plan the feature, but show me three ways to do the realtime part." → Use `plan` as the spine, drop an `options` block inside section 02 (data flow) showing the alternatives for the contested decision.
- **explainer + diagram.** "Explain how X works." Most good explainers have one or two diagrams. Use `explainer` as the spine, embed `diagram`-pattern SVG figures inline.
- **status + diagram.** A post-mortem with an architecture diagram showing where the failure was. Use `status` as the spine, embed a `diagram` in the root-cause section.
- **editor + options.** "Help me pick which of these features to cut, and show me what the cut and remaining feature lists look like." → An `editor` (drag-to-cut) whose export emits two markdown lists.

When combining, pick the **primary** pattern by which one most determines the overall shape of the file. The secondary pattern contributes a section or a component, not the whole structure. Read both playbooks; resolve conflicts in favor of the primary.

## Anti-patterns

- **Markdown-in-a-`<div>`.** If your `<body>` is one big `<div>` of paragraphs and headers, you've produced HTML but not a useful artifact. The content has no shape — use markdown.
- **Generic-LLM aesthetics.** Avoid the gradient hero, the floating shadowed card, rainbow emoji bullets. These telegraph "AI made this" and undermine authority. Use the base template's restrained tokens; don't reach for decoration.
- **No export on an editor.** An interactive artifact without a "copy as ..." button is half-built. The user can't get the work back out.
- **Frameworks.** No React, no Vue, no Tailwind, no Bootstrap. The point is *no build step*. (Exception: if the user explicitly asks for "a React component", that's a different request — not this skill.)
- **External assets.** System fonts, inline SVG, no `<link rel="stylesheet">`, no `<img src="https://...">`. Render offline.
- **Overlong files.** A good artifact in any of these patterns is 200–700 lines of HTML. Past 1000, you're building a website when the user wanted a page.

## Self-check before declaring done

1. Does it open standalone with no network?
2. Is there a "Prompt:" block near the top?
3. Does the content have shape — grid, timeline, table, diagram — not stacked paragraphs?
4. If interactive: working export button? Export format chosen *before* the UI?
5. Mobile-readable (columns collapse, type scales)?
6. Kebab-case filename, descriptive of the topic?
7. Did you read the pattern playbook(s)? If no, go back.
8. Did you keep the locked look — cream canvas, coral accent, EB Garamond display, Inter body — straight from `base.html`? Compare visually against `gallery/look.html` if unsure.

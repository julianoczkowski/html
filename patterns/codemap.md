---
name: codemap
description: Draw an unfamiliar codebase or module as a single HTML page with an SVG diagram of boxes-and-arrows, a hot-path highlight, an entry-point list, and a project-specific glossary. This skill should be used when the user asks "help me understand this codebase", "what does this package do", "map this module", "where does X get called from", or any onboarding-style code-exploration request. Software-specific.
---

# codemap

The reader is new to this code. Their question is the same one everyone has on day one: "where do I start?" A map answers better than a README does.

## Read first

Open `../templates/base.html`. Use its `.callout.info` for the TL;DR. Read `patterns/diagram.md` for SVG conventions — this pattern's diagram follows them.

## Structure

```
<kicker>Codemap · <package>
<h1><Package name in plain words>
<prompt block>

<TL;DR callout>           ← what this package does, who calls into it
<svg diagram>             ← boxes (modules) and arrows (calls/data flow)
                            with the HOT PATH highlighted in accent color
<entry points list>       ← the 3-5 functions/exports a newcomer should read first,
                            with file:line and a one-sentence "this is where X happens"
<glossary>                ← 5-10 domain terms with one-line definitions,
                            for the words that recur in the code
```

## Rules

1. **Diagram is hand-rolled SVG, never an ASCII box.** The reader is looking at this to *see* structure; markdown art defeats the purpose. Read `diagram.md` for the conventions.
2. **Highlight one path through the diagram in accent color.** The "hot path" — the most common request flow. New readers anchor on it; everything else is variation.
3. **Boxes are modules/files; arrows are calls.** Label arrows with the operation ("creates", "validates", "publishes") when not obvious. Dashed arrows = async or fire-and-forget.
4. **Entry points are real `file:line` references.** Not "see the auth module" — `packages/api/src/auth/login.ts:42`. The reader will cmd-click; make that work.
5. **Glossary entries are for *this codebase's* vocabulary.** Don't define "request" or "controller." Do define the project-specific nouns ("workspace", "thread", "channel") because newcomers conflate them with their generic meaning.
6. **Keep it under one screen of diagram.** If the module is too big for one diagram, draw the top-level map and link to sub-maps. Don't cram 40 boxes into one SVG.

## Pattern-specific CSS (extends base utility kit)

```css
.diagram-wrap { margin: 2em 0; padding: 24px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); }
.diagram-wrap svg { display: block; max-width: 100%; height: auto; }
.entry { display: grid; grid-template-columns: minmax(160px, auto) 1fr; gap: 8px 16px; padding: 10px 0; border-bottom: 1px solid var(--line); }
.entry code { font-size: 0.88rem; color: var(--accent); }
.entry .desc { color: var(--ink-muted); }
dl.glossary { display: grid; grid-template-columns: minmax(120px, auto) 1fr; gap: 6px 16px; margin: 1em 0; }
dl.glossary dt { font-weight: 600; }
dl.glossary dd { margin: 0; color: var(--ink-muted); }
```

## Export

No export — this is a reading artifact, not an editing one.

## Anti-patterns

- **A diagram that's just the directory tree.** Directory structure is rarely the same as call structure; draw the calls.
- **Highlighting too many things.** "Hot path" means *one* path. If multiple paths feel hot, pick the most common request.
- **Entry points list with no file paths.** The reader can't navigate from prose.
- **Generic glossary entries.** Only define terms that mean something specific in *this* codebase.

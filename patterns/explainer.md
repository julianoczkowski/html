---
name: explainer
description: Build a layered explainer as a single HTML page with TL;DR, collapsible deep-dives, tabs for parallel content (languages, configs, formats), and a hover-glossary. This skill should be used when the user asks "explain how X works", "teach me Y", "write an explainer for Z", or wants to onboard themselves or others onto a non-trivial concept. Works for technical concepts, scientific ideas, policies and procedures, financial instruments, legal concepts — anything that benefits from layered depth instead of a linear read.
---

# explainer

An explainer with collapsible sections, tabs, and a glossary in the margin reads completely differently from the same words dumped linearly. The scaffolding *is* the value — it lets the reader skim, drill in where curious, and skip what they already know.

## Read first

Open `../templates/base.html`. Use its `.callout.info` for the TL;DR, native `<details>` styled by the base for collapsibles, and the tab pattern below.

## Structure

```
<kicker>Explainer · <topic or domain>
<h1><The concept or feature>
<prompt block>

<TL;DR callout>       ← 2-3 sentences. The reader who reads nothing else gets this.
<sections>            ← each section is a layered explanation:
  <h2>step or concept name
  <p>the surface explanation
  <details>          ← "Why this works" / "Edge cases" / "When this breaks"
<tabbed content>      ← parallel content: same idea in different forms
<comparison table>    ← if the concept has variants, contrast them
<FAQ>                 ← real questions, in <details> elements
<glossary>            ← domain terms, with hover-links from the body
```

## Rules

1. **TL;DR is mandatory and first.** Three sentences max. Use the base's `.callout.info`. The reader with 30 seconds leaves the page knowing the core idea.
2. **Collapsibles use `<details>`.** Native HTML, accessible, no JS. Default-closed for "going deeper"; default-open for "you must read this to continue."
3. **Tabs use radio-buttons + CSS, no JS.** Pattern below. The HTML stays semantic.
4. **Glossary terms hover-link from the body.** `<span class="gloss" data-def="...">term</span>`. On hover, the definition floats above. The glossary section at the bottom is the canonical list.
5. **Examples are real and short.** A 50-line code block is a tutorial, not an explainer. Keep examples focused on the single idea being explained — 3 to 15 lines of code, or 2–3 sentences of example prose.
6. **An interactive demo earns its place only if it teaches something static can't.** A consistent-hashing ring you can add/remove nodes from is worth building. A button that toggles a class isn't.
7. **FAQ entries are real questions, not made-up.** Generate them from the parts of the explanation that felt fragile or hand-wavy. If you wrote "we'll discuss this later" anywhere, the FAQ is where "later" happens.

## Pattern-specific CSS (extends base utility kit)

```css
.tabs { margin: 1.4em 0; }
.tabs input[type="radio"] { display: none; }
.tabs .tablist { display: flex; gap: 4px; border-bottom: 1px solid var(--line); }
.tabs .tablist label { padding: 8px 14px; cursor: pointer; font-size: 0.88rem; color: var(--ink-muted); border-bottom: 2px solid transparent; margin-bottom: -1px; }
.tabs .tablist label:hover { color: var(--ink); }
.tabs .panel { display: none; padding: 16px 0; }
.tabs input[type="radio"]:checked + label { color: var(--ink); border-bottom-color: var(--accent); }
.gloss { border-bottom: 1px dotted var(--ink-faint); cursor: help; position: relative; }
.gloss:hover::after, .gloss:focus::after {
  content: attr(data-def);
  position: absolute;
  bottom: 100%; left: 0;
  background: var(--ink); color: var(--bg);
  padding: 8px 12px; border-radius: var(--radius);
  font-size: 0.85rem; white-space: normal; width: 280px; z-index: 10;
  margin-bottom: 6px;
}
dl.glossary { display: grid; grid-template-columns: minmax(120px, auto) 1fr; gap: 8px 16px; }
dl.glossary dt { font-weight: 600; }
dl.glossary dd { margin: 0; color: var(--ink-muted); }
```

(`.callout.info` and `<details>` styling come from the base.)

## Tabs HTML pattern (no JS)

```html
<div class="tabs">
  <div class="tablist">
    <input type="radio" name="t" id="t-a" checked>
    <label for="t-a">YAML</label>
    <input type="radio" name="t" id="t-b">
    <label for="t-b">TOML</label>
  </div>
  <div class="panel" data-for="t-a">...YAML example...</div>
  <div class="panel" data-for="t-b">...TOML example...</div>
</div>
<style>
  #t-a:checked ~ .panel[data-for="t-a"],
  #t-b:checked ~ .panel[data-for="t-b"] { display: block; }
</style>
```

(The inputs sit as siblings of the panels so the `:checked ~` selector matches without JS.)

## Cross-domain fit

- **Technical concept:** *"How does rate limiting work in this repo"* — request-path collapsibles, tabs for nginx/redis/app-level config, glossary of project-specific terms.
- **Scientific idea:** *"Explain how mRNA vaccines work"* — collapsibles for each stage (entry, translation, response), tabs comparing mRNA vs viral-vector, glossary of immunology terms.
- **Financial instrument:** *"Explain how convertible notes work"* — collapsibles for each clause type, tabs for SAFE vs convertible vs equity, glossary of cap-table terms.
- **Policy / procedure:** *"How does our expense-approval flow work"* — collapsibles for each step, tabs for self-serve vs manager-approval, glossary of system names.

Same scaffolding; the depth of the layers and the choice of tabs adapt to the domain.

## Export

No export — this is a reading artifact. Optional: a "Copy as markdown" button using the base's `button.export` style, for someone who wants to paste a flattened version into a wiki.

## Anti-patterns

- **Linear wall of text with no structural layering.** If the explainer reads the same as the markdown version would, you've wasted the medium.
- **TL;DR longer than the explanation.** The TL;DR is for the skimmer; if they need more, they read on. 3 sentences is plenty.
- **Tabs with three tabs where one is empty or trivially different.** Use a tab only when there are real alternatives.
- **Glossary defining words anyone reading already knows.** Define the *project-* or *domain-specific* vocabulary only.
- **"Interactive" demo that's a button toggling a class.** The demo should teach something static can't, or it's decoration.
- **No FAQ.** Means you haven't asked yourself "what would the reader still wonder about?"

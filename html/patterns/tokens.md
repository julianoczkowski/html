---
name: tokens
description: Render a design system as a single HTML page with live color swatches, type ramp, spacing tokens, and a component contact sheet. This skill should be used when the user asks to "render our design tokens", "show me the design system", "make a style guide", "lay out the component variants", or pulls tokens from a config file and wants to see them. Software/design-specific.
---

# tokens

HTML is the medium your design system ships in, so it's the natural surface for talking about it. Render tokens as the actual things they describe: colors as swatches, type as text at that size, spacing as visible boxes.

## Read first

Open `../templates/base.html`. **Override its CSS variables** with the design system's actual tokens — this page should look like the system it documents.

## Structure

```
<kicker>Design system · <project>
<h1><System name>
<prompt block>

01 Colors          ← swatches: chip + hex + token name + computed value, click-to-copy
02 Typography      ← type ramp at real sizes, with line-height/weight/intended use
03 Spacing         ← visualized as filled boxes at actual size, labeled
04 Radii & shadows ← small live samples of each
05 Components      ← contact sheet: every variant of one component laid out in a grid
```

## Rules

1. **Swatches show color, hex, token name, and (where useful) contrast indicator.** Click-to-copy on the swatch — the swatch *is* the copyable thing.
2. **Type ramp renders at the actual size.** Don't write "H1: 32px"; render the H1 *at 32px* with "32px / 1.25 / 600" as a caption beside it.
3. **Spacing shown as filled boxes at the actual size.** A token `--space-4: 16px` is a 16px×16px filled square next to its name. The page becomes a visual ruler.
4. **Component contact sheet: every variant of one component on one page.** Sizes × intents × states. Use the base's `.grid.cols-3` or `.cols-4`. State label below each.
5. **The page itself uses the tokens it documents.** This is the discipline that makes the artifact real — if the swatches look wrong on the page, the tokens are wrong.
6. **Copy buttons everywhere.** Every token name, hex code, and CSS variable should be copyable with a click. The page is a lookup tool first; optimize for "I need this value, get it into my clipboard."

## Pattern-specific CSS (extends base utility kit)

```css
.swatches { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 12px; margin: 1.4em 0; }
.swatch { border: 1px solid var(--line); border-radius: var(--radius); overflow: hidden; cursor: pointer; transition: transform 0.1s; }
.swatch:hover { transform: translateY(-2px); }
.swatch .chip-color { height: 80px; }
.swatch .meta { padding: 8px 10px; font-family: ui-monospace, monospace; font-size: 0.78rem; }
.swatch .meta .name { color: var(--ink); font-weight: 600; }
.swatch .meta .hex { color: var(--ink-muted); }
.typeramp .row { display: grid; grid-template-columns: 1fr 200px; gap: 24px; align-items: baseline; padding: 16px 0; border-bottom: 1px solid var(--line); }
.typeramp .row .specimen { color: var(--ink); }
.typeramp .row .caption { font-family: ui-monospace, monospace; font-size: 0.82rem; color: var(--ink-muted); }
.spacing-row { display: flex; align-items: center; gap: 16px; padding: 8px 0; }
.spacing-row .box { background: var(--accent); border-radius: 2px; }
.spacing-row .label { font-family: ui-monospace, monospace; font-size: 0.88rem; }
.spacing-row .label b { color: var(--ink); }
.spacing-row .label span { color: var(--ink-muted); margin-left: 8px; }
.contact-sheet { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 24px; margin: 1.4em 0; padding: 24px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); }
.contact-sheet .cell { display: flex; flex-direction: column; align-items: flex-start; gap: 8px; }
.contact-sheet .cell .label { font-family: ui-monospace, monospace; font-size: 0.72rem; color: var(--ink-faint); text-transform: uppercase; letter-spacing: 0.06em; }
```

## Export

A "Copy all tokens as CSS" button at the bottom (use base's `.export-primary`) emitting a `:root { --token: value; }` block. Optional secondary: "Copy as JSON" for tools that consume design tokens that way.

## Anti-patterns

- **Color names with no values.** The reader needs the hex; the name is for reference, not lookup.
- **Type ramp written as a list of font sizes.** Render the text at the size. The whole point is to see it.
- **Component sheet with one variant per component.** The point is *variation* — show all sizes, states, intents on the same page.
- **Page that doesn't use its own tokens.** If the page is in default browser styles but the swatches show a designed palette, you've made a list, not a system.

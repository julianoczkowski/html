---
name: diagram
description: Produce hand-rolled inline SVG diagrams — flowcharts, architecture sketches, process maps, data flows, illustrative figures — in a single HTML file. This skill should be used when the user asks for a diagram, flowchart, architecture sketch, system diagram, process map, or illustrative figures for an article — and would otherwise get a Mermaid block. Works for software architecture, business processes, scientific concepts, organizational charts, anything where boxes and arrows beat prose.
---

# diagram

Inline SVG gives the agent a real pen. Use it instead of reaching for Mermaid or PlantUML. Mermaid is fine for a quick sketch but produces generic-looking output, and the user can't tweak it without rerunning a build. Hand-rolled SVG is a few extra lines and the result is a real figure.

## Read first

Open `../templates/base.html`. Use its color tokens — `var(--ink)`, `var(--line-strong)`, `var(--accent)` — so the diagram matches the page.

## Two sub-modes

**A. Figure sheet** — a set of illustrative diagrams (e.g. for a blog post, lecture, report), each in its own `<figure>` with a caption.

**B. Single annotated diagram** — one large diagram (flowchart, architecture, process map) with click-to-reveal annotations on each node.

## Structure — Figure sheet

```
<kicker>Figures · <post or doc>
<h1><Post title or topic>
<prompt block>

<figure>            ← each SVG with a caption beneath
<figure>
<figure>
...
```

## Structure — Single annotated diagram

```
<kicker>Diagram · <system or process>
<h1><What this diagram shows>
<prompt block>

<svg>                ← one large SVG, nodes are clickable
<annotation panel>   ← updates when a node is clicked, showing details
```

## Rules — drawing

1. **Sensible viewBox, scale to fit.** `viewBox="0 0 800 500"` then `style="width: 100%; height: auto"`. Never fixed pixel dimensions — the page must work on mobile.
2. **Boxes have rounded corners (4–8px) and 12–16px padding.** Sharp corners read as harsh; over-rounded reads as toy. Aim for "engineering drawing", not "kindergarten."
3. **Arrows via `<marker>`.** Define an arrowhead marker once at the top, reuse it. Don't draw triangles by hand.
4. **Solid vs dashed has meaning.** Solid = sync / synchronous / direct. Dashed = async / fire-and-forget / indirect. State this in a small legend below the diagram.
5. **Label every arrow.** A box-and-arrow diagram with unlabeled arrows is half-done. The verb on the arrow is what makes the diagram readable ("creates", "validates", "publishes to", "reviews", "approves").
6. **Use the base palette.** `var(--ink)` for text, `var(--line-strong)` for box strokes, `var(--accent)` to highlight the hot path or current focus. Don't introduce new colors unless the diagram has more than one semantic axis (e.g. one color for "owned by team A", another for "owned by team B").
7. **Text inside SVG uses `font-family: inherit`.** So it picks up the page font and the diagram looks like part of the document, not pasted in.

## Rules — annotated mode interactivity

- Clickable nodes have `cursor: pointer` and `tabindex="0"`.
- Use `aria-controls` to point at the annotation panel so it's keyboard-accessible.
- Annotation panel updates via a small JS event listener; no framework.

## Pattern-specific CSS (extends base utility kit)

```css
figure { margin: 2em 0; padding: 20px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); }
figure svg { display: block; width: 100%; height: auto; }
figcaption { margin-top: 12px; font-size: 0.9rem; color: var(--ink-muted); font-style: italic; }
.legend { font-size: 0.82rem; color: var(--ink-muted); margin-top: 12px; display: flex; flex-wrap: wrap; gap: 16px; }
.legend span { display: inline-flex; align-items: center; gap: 6px; }
.legend i { display: inline-block; width: 18px; height: 2px; }
.annotated { display: grid; grid-template-columns: 2fr 1fr; gap: 24px; margin: 2em 0; }
@media (max-width: 720px) { .annotated { grid-template-columns: 1fr; } }
.annotated .diagram { padding: 20px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); }
.annotated .panel { padding: 20px; background: var(--code-bg); border-radius: var(--radius); min-height: 200px; }
.annotated .panel h4 { margin-top: 0; }
.annotated .panel .empty { color: var(--ink-faint); font-style: italic; }
.annotated svg .node { cursor: pointer; }
.annotated svg .node:hover rect, .annotated svg .node:focus rect { stroke-width: 2; }
.annotated svg .node.active rect { fill: var(--accent-soft); stroke: var(--accent); stroke-width: 2; }
```

## SVG arrowhead marker (copy verbatim)

```html
<defs>
  <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"/>
  </marker>
</defs>
```

Use `marker-end="url(#arrow)"` on `<line>` or `<path>`, and set `stroke="currentColor"` so the arrowhead matches the line color.

## Cross-domain fit

- **Software architecture:** boxes are services, arrows are RPC calls; solid = request/response, dashed = async events.
- **Business process:** boxes are roles or stages, arrows are handoffs; solid = required, dashed = optional fast-path.
- **Scientific concept:** boxes are entities or states, arrows are transformations; solid = forward reaction, dashed = reverse.
- **Org chart:** boxes are roles, arrows are reporting lines; solid = direct, dashed = dotted-line.
- **Customer journey:** boxes are touchpoints, arrows are transitions; solid = primary path, dashed = drop-off recovery.

## Export

Figure sheet mode: each `<figure>` has a small "Copy SVG" button (use base's `.export` style) that emits just that SVG's source — so the user can paste it into a blog post or doc.

Annotated mode: no export; the page is the artifact.

## Anti-patterns

- **Mermaid blocks.** The whole point of this pattern is *not* Mermaid.
- **SVG with fixed pixel dimensions** that overflow on mobile.
- **A 40-node spaghetti diagram.** If you need 40 nodes, you need two diagrams.
- **Decorative gradients and drop shadows.** Engineering drawings, not landing-page hero art.
- **Unlabeled arrows.** Always state what flows along the arrow.
- **Color-coding with no legend.** If you used three colors, three colors are explained in the legend.

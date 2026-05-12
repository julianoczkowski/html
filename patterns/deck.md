---
name: deck
description: Build a single-file HTML slide deck navigable with arrow keys, one slide per section, no build step. This skill should be used when the user asks for slides, a deck, a presentation, or wants to take prose (a Slack thread, a design doc, meeting notes, a research summary) and turn it into something they can arrow-key through in a meeting. Works for any kind of presentation — engineering review, sales pitch, conference talk, lecture, all-hands.
---

# deck

A handful of `<section>` tags and twenty lines of JS is a slide deck. The user points at a Slack thread or a doc and gets something they can arrow-key through in a meeting. No Keynote, no PowerPoint, no Reveal.js install.

## Read first

Open `../templates/base.html`. The deck overrides much of the base's layout (body becomes full-viewport, no wrap), but keeps the color tokens and typography.

## Structure

```
<head>             ← title, no scroll on body
<body>
  <main>
    <section class="slide" data-slide="1">  ← one slide per section
      <h2>             ← slide title (large)
      content          ← keep it sparse: 3-5 bullets max, or one big idea
    </section>
    <section class="slide" data-slide="2">...</section>
    ...
  </main>
  <nav class="deck-nav">    ← prev / counter / next at the bottom
  <script>          ← arrow-key handlers, slide swap, overview grid
</body>
```

## Rules

1. **One idea per slide.** If a slide has more than 5 bullets or more than a paragraph of body text, split it. The reader is in a meeting and can only absorb one thing at a time.
2. **Big type.** Title is 3–4rem, body is 1.4rem. The slide is on a screen, possibly a TV, possibly to people not in front of the laptop. Default body text is wrong for a deck.
3. **Centered, generous whitespace.** Each slide is its own viewport. Don't fill edge-to-edge.
4. **Arrow keys, space, and J/K all advance.** Page Up / Page Down go back. ESC shows an overview grid. Home/End jump to first/last. These are the conventions users have from other deck tools.
5. **Slide counter visible.** "3 / 12" in the nav bar. The reader needs to know how much is left.
6. **Speaker notes in `<aside hidden>`.** Toggle with N. Don't print them on the slide.
7. **Real HTML elements per slide.** A code-walkthrough slide has a real `<pre>`. A diagram slide has a real `<svg>`. A quote slide has a real `<blockquote>`. The slide system is just navigation; the content underneath is HTML.

## Pattern-specific CSS (overrides base layout)

```css
body { overflow: hidden; height: 100vh; }
main.wrap { max-width: none; margin: 0; padding: 0; height: 100vh; }
.slide {
  position: absolute; inset: 0;
  display: flex; flex-direction: column; justify-content: center; align-items: center;
  padding: 60px 80px;
  opacity: 0; pointer-events: none;
  transition: opacity 180ms ease;
  text-align: center;
}
.slide.active { opacity: 1; pointer-events: auto; }
.slide h2 { font-size: 3.2rem; line-height: 1.15; margin: 0 0 0.6em; max-width: 18ch; }
.slide ul, .slide ol { font-size: 1.4rem; line-height: 1.5; text-align: left; max-width: 32ch; }
.slide ul li, .slide ol li { margin: 0.5em 0; }
.slide pre { font-size: 1.1rem; max-width: 80ch; text-align: left; }
.slide blockquote { font-size: 1.8rem; font-style: italic; max-width: 28ch; border-left: 4px solid var(--accent); padding-left: 24px; text-align: left; }
.slide .kicker { font-size: 0.85rem; margin-bottom: 1em; }
.deck-nav {
  position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%);
  display: flex; align-items: center; gap: 16px;
  background: color-mix(in srgb, var(--surface) 80%, transparent);
  backdrop-filter: blur(8px);
  padding: 8px 16px; border-radius: 999px; border: 1px solid var(--line);
  font-size: 0.85rem;
}
.deck-nav button { appearance: none; border: 0; background: transparent; cursor: pointer; color: var(--ink); font: inherit; padding: 4px 8px; }
.deck-nav .counter { font-variant-numeric: tabular-nums; color: var(--ink-muted); }
.overview {
  position: fixed; inset: 0; background: color-mix(in srgb, var(--bg) 95%, transparent);
  backdrop-filter: blur(12px);
  display: none; padding: 40px; overflow-y: auto;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 16px;
}
.overview.active { display: grid; }
.overview .thumb { aspect-ratio: 16/10; border: 1px solid var(--line); border-radius: var(--radius); background: var(--surface); padding: 16px; font-size: 0.8rem; cursor: pointer; }
.overview .thumb h3 { font-size: 0.95rem; margin: 0 0 6px; }
.overview .thumb.current { border-color: var(--accent); border-width: 2px; }
```

## JS skeleton (paste verbatim, adjust as needed)

```html
<script>
  const slides = [...document.querySelectorAll('.slide')];
  let i = 0;
  function go(n) {
    i = Math.max(0, Math.min(slides.length - 1, n));
    slides.forEach((s, idx) => s.classList.toggle('active', idx === i));
    document.querySelector('.counter').textContent = `${i + 1} / ${slides.length}`;
  }
  document.addEventListener('keydown', (e) => {
    if (['ArrowRight', ' ', 'j', 'PageDown'].includes(e.key)) go(i + 1);
    if (['ArrowLeft', 'k', 'PageUp'].includes(e.key)) go(i - 1);
    if (e.key === 'Escape') document.querySelector('.overview').classList.toggle('active');
    if (e.key === 'Home') go(0);
    if (e.key === 'End') go(slides.length - 1);
  });
  go(0);
</script>
```

## Cross-domain fit

- **Engineering review:** title slide, problem slide, options slide (3 columns), recommendation slide, implementation slide, risks slide. ~8 slides.
- **Sales pitch:** title slide, the customer's pain, our solution (one big visual), proof points (3 slides), pricing, call to action. ~10 slides.
- **Conference talk:** title, the question that motivates the talk, three sections of body (with section-divider slides between), conclusion, contact info. ~15 slides.
- **All-hands update:** title, the headline number, what shipped (3 slides), what's next (1 slide), open Q&A.

## Export

No export — the file *is* the deck. The user opens it and presents. Optional: a print stylesheet that puts one slide per page so it can be saved as PDF via the browser's print dialog.

## Anti-patterns

- **Slides full of bullets at body-text size.** That's a doc with page breaks, not a deck.
- **Tiny code samples no one can read from the back of the room.**
- **Decorative slide transitions.** A 180ms fade is enough; sliding/zooming pulls focus from the content.
- **Two ideas on one slide because you have lots of content.** Add a slide; they're free.
- **No section dividers in a long deck.** A 20-slide deck needs visual breath every 5–6 slides.

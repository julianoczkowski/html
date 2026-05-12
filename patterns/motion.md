---
name: motion
description: Build a throwaway HTML page that demonstrates a UI motion, transition, or short click-through with real interaction so the reviewer can feel it. This skill should be used when the user asks to "prototype an animation", "show me how this transition would feel", "mock up the click-through for X flow", or when description in prose would fail to convey timing, easing, or interaction quality. Software/UI-specific.
---

# motion

Motion can't be described, only felt. A static screenshot or a paragraph of prose loses the thing that actually matters: how it feels at 250ms vs 400ms, whether the easing reads as snappy or sluggish. A throwaway page with the real animation answers the question in five seconds.

## Read first

Open `../templates/base.html`. The motion page uses much of the base for typography but adds a custom stage area.

## Two sub-modes

**A. Animation sandbox** — one transition in isolation, with controls.

**B. Clickable flow** — 3–6 screens linked together by buttons that simulate the real interaction.

## Structure — Animation sandbox

```
<kicker>Motion · <project>
<h1><Animation name>
<prompt block>

<stage>             ← the animation itself, large and centered
<controls>          ← sliders for duration, easing dropdown, "play" button to retrigger
<spec readout>      ← the current values as CSS, copyable
                      (transition: transform 300ms cubic-bezier(0.4, 0, 0.2, 1))
```

## Structure — Clickable flow

```
<kicker>Motion · <project>
<h1><Flow name>
<prompt block>

<flow nav>         ← step indicator: 1 → 2 → 3 → 4
<viewport>         ← single visible screen at real device dimensions
                     screens swap in with real transition timing
<actions>          ← the buttons inside the viewport advance the flow
<flow controls>    ← prev / next at the bottom, "reset" link
```

## Rules

1. **Real CSS transitions, not GIFs or screenshots.** This is the entire point — the reader needs to scrub a slider and see the change.
2. **Sandbox mode: spec readout is copyable.** The user is tuning so they can paste the final values into the codebase. Make that paste one click.
3. **Flow mode: respect the platform's real dimensions.** A phone flow renders in a phone-sized frame (375×812). A desktop flow renders at desktop sizes. Don't scale; the geometry is part of the feel.
4. **Don't overbuild.** Three screens linked by `click → fade → swap` is enough to feel a flow. Eight screens with full content is a different project.
5. **One transition per page.** If the user wants to compare two easings, render them side-by-side as two stages on one page. Don't bury comparison in a dropdown.
6. **Default to `prefers-reduced-motion`.** Wrap the animations in `@media (prefers-reduced-motion: no-preference)`. The user tuning the motion can disable this temporarily.

## Pattern-specific CSS (extends base utility kit)

```css
.stage { padding: 60px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); display: flex; align-items: center; justify-content: center; min-height: 280px; margin: 2em 0; }
.controls { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 16px 24px; padding: 20px; background: var(--code-bg); border-radius: var(--radius); margin: 1em 0; }
.controls label { display: flex; flex-direction: column; gap: 6px; font-size: 0.85rem; color: var(--ink-muted); }
.controls input[type="range"], .controls select { width: 100%; }
.spec { font-family: ui-monospace, monospace; font-size: 0.88rem; background: var(--code-bg); padding: 14px 16px; border-radius: var(--radius); position: relative; }
.spec button.export { position: absolute; top: 8px; right: 8px; padding: 4px 10px; font-size: 0.8rem; }
.flow-nav { display: flex; align-items: center; gap: 8px; margin: 1em 0; font-size: 0.85rem; color: var(--ink-muted); }
.flow-nav .step { width: 24px; height: 24px; border-radius: 50%; border: 1px solid var(--line-strong); display: inline-flex; align-items: center; justify-content: center; font-weight: 600; }
.flow-nav .step.active { background: var(--accent); border-color: var(--accent); color: white; }
.viewport { width: 375px; max-width: 100%; height: 600px; margin: 1em auto; border: 1px solid var(--line-strong); border-radius: 16px; overflow: hidden; background: white; position: relative; }
.viewport .screen { position: absolute; inset: 0; transition: opacity 240ms ease, transform 240ms ease; }
.viewport .screen[hidden] { display: block; opacity: 0; transform: translateX(20px); pointer-events: none; }
```

## Export

**Sandbox mode**: a "Copy CSS" button on the spec readout that emits the exact `transition` / `animation` declaration with the current tuned values.

**Flow mode**: no export — the artifact is the demo. The user shares the file or screencasts from it.

## Anti-patterns

- **A CSS animation defined with no way to retrigger.** Add a "play" button — the reviewer wants to watch it three times.
- **Sliders that don't update the running animation live.** The whole point is real-time tuning.
- **A "flow" that's static screenshots with prev/next and no transitions.** That's a PDF.
- **Hardcoded values everywhere.** Use CSS custom properties so the controls can rewrite them via JS in one line.

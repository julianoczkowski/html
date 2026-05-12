---
name: options
description: Generate 2-4 candidate approaches to a problem side-by-side in a single HTML file, each with the relevant details, trade-offs, and a recommendation. This skill should be used when the user asks "show me a few ways to do X", "what are my options for Y", "compare approaches to Z", or any request for trade-off analysis before committing to a path — works equally well for code architectures, vendor choices, study designs, marketing strategies, hiring approaches, or any decision with multiple viable paths.
---

# options

Two to four candidates. The same problem solved different ways. Trade-offs paired side-by-side. A recommendation at the bottom. The reader scans, points at one, and you commit.

This is the highest-leverage pattern in the bundle. The wall-of-text alternative — three sequential markdown sections — is worse, because the reader can't hold all three in their head at once.

## Read first

Open `../templates/base.html`. Use its `.grid.cols-2` or `.grid.cols-3` layout, its `.card` class for each option, its `.callout.info` for the recommendation.

## Structure

```
<kicker>Options · <topic>
<h1>Three ways to <do the thing>
<prompt block>

<grid of N cards>     ← each card is one approach
  approach 01
    name + 1-line summary
    body content (real, specific — code, plan, design, whatever fits)
    pros table | cons table  (two-column, paired)
    footer chips: the consistent set of axes you're comparing on
  approach 02
  approach 03

<recommendation callout>  ← which one and why, 2-3 sentences max
```

## Rules

1. **2 to 4 cards. Three is the sweet spot.** Two feels thin; five is too many to compare at a glance.
2. **Same axes for every option.** If option 01 lists cost, every option lists cost. The reader is comparing, and comparing requires identical axes. Pick the axes that matter for the decision, declare them once, fill them in for every option.
3. **Real, specific content.** The body of each card should be concrete enough to evaluate — actual code, an actual venue with an address, the actual study design, the actual budget split. Not "approach A is the modern one." If you'd need a follow-up question to evaluate it, the option isn't specific enough yet.
4. **Pros and cons in a two-column grid, paired by row.** First pro paired with first con on the same row. This forces honesty (each pro implies a trade-off) and reads faster than two separate lists.
5. **The recommendation is mandatory.** Don't punt with "it depends" or "any of these would work." Pick one, name it, give 2–3 sentences of why. The reader can override your call — they need a default to push against.
6. **Numbered headings: 01, 02, 03.** Small grey number left of the option name. Mimics Thariq's house style and scans well.

## Pattern-specific CSS (extends base utility kit)

```css
.tradeoffs { display: grid; grid-template-columns: 1fr 1fr; gap: 8px 16px; margin: 1em 0; font-size: 0.92rem; }
.tradeoffs .head { font-size: 0.72rem; letter-spacing: 0.08em; text-transform: uppercase; color: var(--ink-faint); }
.tradeoffs .pro::before { content: "✓ "; color: var(--good); }
.tradeoffs .con::before { content: "✗ "; color: var(--bad); }
.axes { display: flex; flex-wrap: wrap; gap: 6px 14px; font-size: 0.82rem; color: var(--ink-muted); margin-top: 1em; padding-top: 1em; border-top: 1px solid var(--line); }
.axes b { color: var(--ink); font-weight: 600; }
```

The base template already provides `.grid.cols-3`, `.card`, `.card .num`, `.card .summary`, and `.callout.info` — use those rather than redefining.

## What "axes" means (the consistent comparison row)

The axes are the named values you tag at the bottom of every card. They're the 3–5 things the reader cares about across all the options. Examples — invent your own per decision:

- **Code:** bundle impact · testability · reuse · SSR-safe
- **Vendor:** monthly cost · setup time · vendor lock-in · SLA
- **Venue:** capacity · price/person · distance from office · catering included
- **Marketing channel:** CPM · audience match · creative effort · measurability
- **Study design:** sample size · time to results · cost · external validity

If you can't think of 3 axes that vary meaningfully across the options, you don't actually have distinct options. Go back to one.

## Export

No export — this artifact exists for *choosing*, not editing. The next prompt the user issues ("let's go with option 02") is the export.

## Anti-patterns

- **Four options where one is a strawman.** If you wouldn't actually recommend an option under any conditions, drop it. The reader knows when an option is filler.
- **Hedged recommendation** ("any of these would work"). The whole point of the artifact is to enable a decision.
- **Different shapes per card.** If card 01 shows the implementation and card 02 shows the cost analysis, the reader can't compare. Use the same content shape across cards.
- **Trade-offs that aren't trade-offs** ("Pro: it works"). Each pro should imply a specific con elsewhere on the same card or on a different card.
- **Generic axes** that don't matter for this decision. "Quality: high" tells the reader nothing. Pick axes the options actually differ on.

## Example openers across domains

To make the cross-domain fit concrete:

- *"Three ways to implement debounced search in our React codebase"* → cards have code snippets; axes are bundle/testability/reuse.
- *"Four venues for the team offsite"* → cards have address, capacity, price; axes are capacity/price/distance/catering.
- *"Two study designs for the retention question"* → cards have method, sample, duration; axes are sample size/time/cost/validity.
- *"Three positioning options for the launch"* → cards have headline, audience, channel mix; axes are audience match/cost/risk/measurability.

Same structure, same playbook, different content.

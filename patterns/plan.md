---
name: plan
description: Produce a thorough plan as a single HTML file with an at-a-glance row, milestones on a timeline, a diagram or mockup section, the risky details called out, and a risk table. This skill should be used when the user asks for a plan, project plan, build plan, technical spec, design doc, launch plan, campaign plan, or "write up how we'd do X". Especially valuable when the plan will be handed off to a different doer — engineer, marketer, designer, contractor, ops lead.
---

# plan

The densest pattern in the bundle. One file that contains everything a different doer needs to start: milestones, a diagram of how it'll work, mockups or sketches of what it'll look like, the risky details called out, a risk table, and the open questions. The reader skims it on a phone; the doer reads it carefully when they pick the work up.

## Read first

Open `../templates/base.html`. Use its `.glance` for the stats row, `.timeline` for milestones, `.section-num` for the section numbers, `.chip` for tags, and tables styled by the base for the risk table.

## Structure (six numbered sections, in this order)

```
<kicker>Plan · <project>
<h1>The thing being planned, in plain words
<prompt block>

<glance>            ← 3-5 small stats: effort · scope · cost · key constraint · etc.

01 Milestones       ← 3-5 slices, each independently reviewable, dated
02 How it works     ← a diagram (SVG) or a sketch of the flow / process / approach
03 Mockups          ← sketches of the user-visible result; for non-UI plans, sample
                      outputs (the slide, the email, the report cover, etc.)
04 The tricky parts ← the 2-3 things most likely to go wrong, with the specific
                      detail that makes them tricky (code snippet, copy paragraph,
                      math, edge case)
05 Risks & mitigations  ← table: risk · severity · mitigation
06 Open questions   ← decisions still needed, with the named decider
```

## Rules

1. **At-a-glance row goes first.** 3–5 numeric facts the reader can absorb in two seconds. *"Effort: ~2 weeks · Surfaces: 3 packages · New tables: 2 · Flag: feature_v1"* for a software plan; *"Duration: 6 weeks · Budget: $40k · Reach: 200k · Channels: 4"* for a marketing plan. This is what the skeptic-skimming-on-a-phone needs.
2. **Milestones are slices, not phases.** Each slice produces something concrete and reviewable. Number them, date them ("Week 1 · Mon–Tue"), tag what's involved. Use the `.timeline` utility from the base.
3. **The "how it works" diagram is hand-rolled SVG, not a Mermaid block.** Boxes with labels, arrows showing flow, callouts on the parts that matter. Caption it.
4. **Mockups are illustrative, not finished.** Just enough fidelity to agree on shape, structure, and tone. For software: HTML mockups of the UI. For marketing: a sample headline + body + CTA in the actual format. For a research plan: a sample table of expected results. Don't ship something pixel-perfect; ship something the reader can react to.
5. **"The tricky parts" is *the risky bits only*.** Don't dump the whole plan. Two or three things where the reader's first attempt would probably be wrong, each with the specific detail (the migration SQL, the rate-limit math, the exact copy of the apology paragraph, the consent script wording).
6. **Risks table has three columns: Risk · Severity · Mitigation.** Severity is a `.chip` (info/warn/bad). Mitigation is concrete (a specific design choice), not "monitor closely."
7. **Open questions name the decider.** *"Decide with: design, before slice 2."* A question without an owner doesn't get answered.

## Pattern-specific CSS (extends base utility kit)

```css
.milestone { border-left: 3px solid var(--line); padding: 0 0 1.4em 20px; margin-left: 8px; position: relative; }
.milestone::before { content: ""; position: absolute; left: -7px; top: 6px; width: 11px; height: 11px; border-radius: 50%; background: var(--accent); border: 2px solid var(--bg); }
.milestone .when { font-size: 0.82rem; color: var(--ink-faint); text-transform: uppercase; letter-spacing: 0.06em; }
.milestone h3 { margin: 4px 0 8px; }
.milestone .tags { margin-top: 8px; display: flex; flex-wrap: wrap; gap: 6px; }
.openq { border: 1px solid var(--line); border-radius: var(--radius); padding: 16px 18px; margin: 12px 0; }
.openq h4 { margin: 0 0 6px; }
.openq .decider { font-size: 0.82rem; color: var(--ink-faint); margin-top: 8px; }
```

(The base already provides `.glance`, `.section-num`, `.chip`, tables, `.timeline`. These pattern-specific styles add only the milestone variant and open-question card.)

## Cross-domain fit

Same structure, different content:

- **Software feature plan:** glance has effort/packages/tables/flag; section 02 is a data-flow SVG; section 03 is HTML mockups of UI; section 04 is migration SQL and the optimistic-update code.
- **Marketing campaign plan:** glance has duration/budget/reach/channels; section 02 is a funnel diagram; section 03 is a sample creative; section 04 is the contested headline copy and the segmentation logic.
- **Research study plan:** glance has duration/sample/cost/IRB-status; section 02 is a flow of participant journey; section 03 is a sample table of results; section 04 is the consent script wording and the analysis plan.
- **Renovation plan:** glance has duration/budget/disruption/permits; section 02 is a floor plan; section 03 is mood-board sketches; section 04 is the load-bearing-wall calculation and the contractor SOW clauses.

The frame holds. The content adapts.

## Export

No export — this is a hand-off document. The reader either approves it, comments on it, or works from it.

## Anti-patterns

- **Plan with no dates.** "Quick to ship" is not a milestone.
- **Risks that are actually tasks.** ("Risk: we need to write the migration." That's a task, not a risk.) Real risks have probability and impact — name what could go wrong, how badly, and what you'll do about it.
- **Mermaid block dumped as text.** Draw it. The agent has SVG.
- **Full code dump in "tricky parts".** The doer can write the easy parts themselves; show only what they're likely to get wrong.
- **Open questions with no owner.** Add "Decide with: ___" or delete the question.
- **Plan that's actually a wish list.** A plan commits to specific moves in a specific order. "We should improve X, fix Y, and grow Z" is a wish list. Convert wishes into slices.

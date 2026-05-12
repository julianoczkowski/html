---
name: status
description: Produce a status report, weekly update, project update, post-mortem, incident report, or retro as a single HTML page with a small chart, color-coded items, callouts, and a timeline. This skill should be used when the user asks for a status report, weekly update, post-mortem, incident report, project update, sprint retro, campaign recap, or any recurring report that benefits from being scannable rather than read linearly. Works for engineering, marketing, ops, research — anywhere a team reports on progress or analyzes what happened.
---

# status

Recurring reports — Monday status, post-mortem, sprint retro, campaign recap — benefit most from a bit of structure and color. A small chart, a colored timeline, status chips on each item. The result: something people actually read instead of skim past in Slack.

## Read first

Open `../templates/base.html`. Use its `.glance`, `.timeline`, `.callout` (warn/bad/info), `.chip` (info/warn/bad/good), and tables.

## Two sub-modes

**A. Update / weekly status** — what shipped, what slipped, the small chart, what's next.

**B. Post-mortem / incident** — minute-by-minute timeline, what happened, root cause, action items.

## Structure — Update mode

```
<kicker>Status · <project or team> · <period>
<h1>Week of <date> (or: <project> · <period>)
<prompt block>

<headline>            ← one-sentence summary up top, bolded, in a .callout.info
<chart>               ← small inline SVG chart (1 metric, 4-8 data points)
<sections>:
  Shipped / Done      ← bulleted with status chips
  Slipped / Delayed   ← bulleted, with why and new ETA
  In progress         ← bulleted, with % complete or expected end date
  Next                ← what's coming next period
<callouts>            ← blockers / asks / decisions needed, pulled out, colored
```

## Structure — Post-mortem mode

```
<kicker>Post-mortem · <severity> · <date>
<h1><Incident name in plain words>
<prompt block>

<glance>             ← duration · impact · severity · status
<summary>            ← 2-3 sentences: what happened, what we did, what's next
<timeline>           ← every event with timestamp, color-coded by phase
<root cause>         ← prose with the specific detail (code snippet, config, etc.)
<contributing factors>
<what worked>        ← honest list, not perfunctory
<what didn't>        ← honest list, internal causes only
<action items>       ← table: action · owner · due · status
```

## Rules

1. **Charts are inline SVG, hand-drawn.** A bar chart with 6 bars is 20 lines of SVG. No Chart.js. Serve *one* metric per chart — don't try to put velocity, burndown, and bug count on the same chart.
2. **Status chips use a fixed vocabulary.** Pick a set and stick to it for the whole document: DONE / WIP / BLOCKED / SLIPPED, or SHIPPED / IN REVIEW / NOT STARTED. Random freeform chips defeat the purpose. Use the base's `.chip.good`, `.chip.info`, `.chip.warn`, `.chip.bad` for color.
3. **Severity chips use a separate scale.** P0/P1/P2 (or HIGH/MED/LOW). Don't conflate status and severity.
4. **Timelines are real.** Use the base template's `.timeline` utility. Each event gets a color class (`.info`, `.warn`, `.bad`, `.good`) by phase: detection (warn), response (info), customer-impact (bad), resolution (good).
5. **Blockers and asks are pulled out as callouts, not buried in bullets.** Don't hide "we need approval on X" in the third paragraph. Put it in a `.callout.warn` or `.callout.bad` near the top. The whole point of the report is to surface what needs attention.
6. **Action items have named owners and dates.** "TBD" in the owner column is the failure mode of every post-mortem. Force a name in every row, or drop the row.
7. **The report is short.** A weekly update is one-screen-scroll. A post-mortem can be longer, but every section header should let the reader decide whether to read the section.

## Pattern-specific CSS (extends base utility kit)

```css
.chart { margin: 1.4em 0; padding: 16px 20px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); }
.chart h4 { margin: 0 0 8px; font-size: 0.85rem; letter-spacing: 0.06em; text-transform: uppercase; color: var(--ink-faint); }
.chart svg { display: block; width: 100%; height: auto; max-height: 200px; }
```

(Everything else — `.glance`, `.callout`, `.chip`, `.timeline`, tables — comes from the base.)

## Cross-domain fit

- **Engineering weekly status:** headline = "Shipped the realtime fan-out; comment threads gated behind flag." Chart = open bugs over 6 weeks. Sections track epics; chips track shipped/wip/blocked.
- **Marketing campaign weekly:** headline = "Launch CTR 3.1%, above 2.4% target." Chart = daily impressions. Sections track channels; chips track on-pace/behind/paused.
- **Research project monthly:** headline = "Recruitment 60% complete, finishing 2 weeks late." Chart = enrollments per week. Sections track study sites; chips track enrolling/full/halted.
- **Incident post-mortem (engineering):** glance = duration/users-affected/severity/status. Timeline = the 4-hour incident.
- **Incident post-mortem (ops / customer support):** glance = duration/tickets-affected/severity/status. Same timeline shape.

## Export

A "Copy as markdown" button — most status reports get pasted into Slack or Notion eventually. Use the base template's `button.export` style. The export emits chips as plain-text suffixes (`Auth migration [DONE]`) and the timeline as a chronological list.

## Anti-patterns

- **A "chart" that's a table of numbers.** If the visual isn't visual, drop it.
- **Wall of bullets with no chips, no callouts, no color.** That's a markdown doc.
- **Action items with TBD owners.** Force a name; if no one will own it, it's not an action.
- **"What didn't work" section that's all external causes.** ("The cloud provider had an outage.") Honest retros name internal contributing factors too — the alert that didn't fire, the runbook that was stale.
- **A weekly that's longer than one screen.** If there's that much to say, you're conflating "status" with "plan." Split them.
- **Severity-as-emoji-spam.** 🔴🔴🔴 is not severity. Use the `.chip` classes.

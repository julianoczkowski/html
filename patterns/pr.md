---
name: pr
description: Produce either an annotated pull request review or an author's PR description as a single HTML file with margin notes, severity tags, and jump links. This skill should be used when the user asks for a code review, PR review, PR description, PR summary, or "write up this change" for either the reviewer or author side of a code-review conversation. Software-specific — for non-code reviews, use status or options.
---

# pr

Two related artifacts, same playbook. Pick a mode at the top.

**Reviewer mode**: annotated diff with margin notes, severity tags, jump links. Easier to scan than scrolling a terminal.

**Author mode**: motivation, before/after, file-by-file tour with the *why*, where to focus the review.

## Read first

Open `../templates/base.html`. Use its `.chip`, `.callout`, and table styles.

## Structure — Reviewer mode

```
<kicker>Code review · <repo or branch>
<h1><PR title>
<prompt block>

<jump nav>             ← bulleted file list, each a link to its diff section
<summary callout>      ← does it do what it says; anything blocking
<diff sections>        ← one per file: header, diff lines, anchored margin notes
<verdict callout>      ← approve / request-changes / comment, with gating items
```

## Structure — Author mode

```
<kicker>Pull request · <repo or branch>
<h1><PR title>
<prompt block>

<motivation>           ← why this change exists (the problem, in plain words)
<before/after grid>    ← two cards: how it works today vs after this PR
<file tour>            ← for each non-trivial file: 1-2 sentences on what changed & why
<review guidance callout>  ← "focus your review on…", "skip the renames in X.ts"
<test plan>            ← what to run locally, what CI covers, what's manual
```

## Rules

1. **Pick one mode at the top.** The kicker text declares it ("Code review" vs "Pull request"). Don't blend.
2. **Diffs are real monospace blocks with red/green line highlighting** — not screenshots, not images. Line numbers in a gutter so margin notes can reference them.
3. **Margin notes have severity.** `blocker` (bad), `nit` (muted), `question` (info), `praise` (good). Use the base's `.chip` color classes plus the mode-specific names below. Color-code them so a reviewer can scan.
4. **Jump links work.** `<a href="#file-foo-ts">` and matching `id`s on each section.
5. **The verdict / review guidance is not optional.** A review without a verdict is unfinished. An author writeup without "focus here" wastes the reviewer's time.

## Pattern-specific CSS (extends base utility kit)

```css
.diff { background: var(--code-bg); border: 1px solid var(--line); border-radius: var(--radius); overflow-x: auto; margin: 1.4em 0; }
.diff-file { display: flex; justify-content: space-between; align-items: center; padding: 8px 14px; background: color-mix(in srgb, var(--ink) 6%, transparent); border-bottom: 1px solid var(--line); font-family: ui-monospace, monospace; font-size: 0.88rem; }
.diff pre { margin: 0; border: 0; border-radius: 0; background: transparent; }
.diff .line { display: grid; grid-template-columns: 48px 1fr; font-family: ui-monospace, monospace; font-size: 0.88rem; line-height: 1.6; }
.diff .line .ln { color: var(--ink-faint); text-align: right; padding-right: 12px; user-select: none; }
.diff .line.add { background: color-mix(in srgb, var(--good) 12%, transparent); }
.diff .line.add .ln { color: var(--good); }
.diff .line.del { background: color-mix(in srgb, var(--bad) 12%, transparent); }
.diff .line.del .ln { color: var(--bad); }
.margin-note { display: grid; grid-template-columns: 100px 1fr; gap: 12px; padding: 10px 14px; border-top: 1px dashed var(--line); font-size: 0.92rem; }
.margin-note .chip { align-self: start; }
.chip.blocker { background: var(--bad-soft); color: var(--bad); }
.chip.nit { background: var(--code-bg); color: var(--ink-muted); }
.chip.question { background: var(--accent-soft); color: var(--accent); }
.chip.praise { background: var(--good-soft); color: var(--good); }
```

For before/after, use `.grid.cols-2` and two `.card` elements from the base.

## Export

**Reviewer mode**: a "Copy as GitHub review" button that emits the comments as `path:line — note` lines, grouped by severity, suitable for pasting into a PR review.

**Author mode**: a "Copy as PR description" button that emits the page as markdown (`## Motivation`, before/after as a table, file tour, test plan) ready to paste into GitHub's PR body.

Use the base's `button.export` (or `.export-primary` if the export is the primary action).

## Anti-patterns

- **Diff as a `<pre>` block with no line numbers and no highlighting.** The whole point is to make it scannable.
- **Margin notes without severity.** The reviewer can't tell what's blocking and what's a nit.
- **Author writeup that's the commit messages concatenated.** The commit log is the *what*; this artifact is the *why* and *where to look*.
- **"LGTM" verdict on a 1000-line PR with no actual review.** If the artifact is empty, don't ship it.

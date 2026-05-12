---
name: editor
description: Build a throwaway interactive HTML editor for the specific thing the user is wrestling with — a triage board, a toggle list, a ranking UI, a tuner — that always ends with an export button emitting markdown, JSON, a diff, or a prompt. This skill should be used when the user says "I need to triage these", "let me toggle some of these", "help me reorder X", "give me a UI to edit Y", or describes a task where dragging/clicking/toggling would be faster than typing prose. Works for prioritization, configuration, ranking, categorization across any domain.
---

# editor

This is the most distinctive pattern in the bundle and the one that fundamentally rewires how the user works with the agent. Instead of describing changes in prose ("move A to Next, move B to Cut, demote C..."), the user gets a small UI built for the thing they're doing, makes the changes by hand, and clicks a button that emits the result as text they can paste back into the conversation.

The artifact is throwaway. The export is the durable output. **The export button is the whole point.** Without it, you've built a toy.

## Read first

Open `../templates/base.html`. Use its `.export-bar` for the sticky bottom bar, `.button.export-primary` for the CTA, and the standard color tokens.

## When the pattern fits

- The user has 10+ items and needs to reorder, categorize, or filter them. (Triage board, kanban, ranking UI, shortlist.)
- The user has a set of toggles or values to tune, where the meaning of one depends on others. (Feature flags with dependencies, config builder, prompt tuner, RSVP tracker.)
- The user will do the work themselves and just needs the right surface to do it on.
- The result is text the user will paste somewhere — a planning doc, a config file, a follow-up prompt, a colleague's inbox.

## When it does NOT fit

- The work is just *deciding*, not *editing*. Use `options` instead.
- The user wants the agent to do the work and just tell them the answer.
- There's no clean "result text" the user would paste anywhere afterward.

## Structure

```
<kicker>Editor · <project> · <what's being edited>
<h1><What this editor does, in plain words>
<prompt block>

<one-line instruction>     ← "Drag tickets across columns. Click a tag to filter."
<the editor itself>        ← the actual UI: board, toggle list, ranking grid, etc.
<sticky export bar>        ← reset link (small, quiet) + export button (big, primary)
                             Emits markdown / JSON / diff / prompt.
```

## Rules

1. **Start with the export format, work backwards.** Before any UI: decide what text the user will copy out. That decides the data model. The UI manipulates the data model; it must always be derivable into the export string.
2. **Initial state comes from the user's input.** If the user pasted a list of items, those items seed the board. If they shared a config, those values seed the toggles. The editor opens *with their data already loaded*, not empty.
3. **Real interactions, not fake ones.** Drag-and-drop uses the HTML5 drag API. Sortable lists really sort. Toggles really toggle. Don't fake interactions with click-to-cycle when drag-and-drop is what the user expects.
4. **State lives in a single in-memory object.** No localStorage unless explicitly requested. One `state = {...}` at the top of the script; one `render()` function that re-renders from it; mutations go through helpers that update `state` and call `render()`.
5. **Export button is the primary CTA.** Use `.export-primary` from the base. Label precisely: "Copy as markdown", "Copy as JSON", "Copy diff", "Copy as prompt" — never just "Export."
6. **Reset link is small and quiet.** A text-button or underline-link, not a big red button. Reset is rare; export is common.
7. **No save, no submit, no server.** The artifact is local. The export *is* the save.

## Pattern-specific CSS (extends base utility kit)

```css
.editor-instr { color: var(--ink-muted); font-size: 0.92rem; margin: 0 0 1.4em; }
.editor-area { background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); padding: 20px; margin: 1.4em 0; }
.reset-link {
  font-size: 0.88rem; color: var(--ink-muted);
  background: none; border: 0; cursor: pointer; padding: 0;
  border-bottom: 1px solid var(--line-strong);
}
.reset-link:hover { color: var(--ink); }

/* Drag-and-drop columns (triage / kanban style) */
.columns { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 16px; }
.column { background: var(--code-bg); border-radius: var(--radius); padding: 14px; min-height: 120px; }
.column h3 { margin: 0 0 12px; font-size: 0.85rem; letter-spacing: 0.08em; text-transform: uppercase; color: var(--ink-muted); display: flex; justify-content: space-between; }
.column h3 .count { background: var(--surface); color: var(--ink); padding: 1px 8px; border-radius: 999px; font-size: 0.78rem; }
.column .card-item { background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); padding: 10px 12px; margin-bottom: 8px; cursor: grab; font-size: 0.92rem; }
.column .card-item:active { cursor: grabbing; }
.column .card-item.dragging { opacity: 0.4; }
.column.drop-target { background: var(--accent-soft); outline: 2px dashed var(--accent); outline-offset: -2px; }

/* Toggle list (config / feature-flag style) */
.toggle-list { display: grid; gap: 8px; }
.toggle-row { display: grid; grid-template-columns: 1fr auto; gap: 12px; align-items: center; padding: 10px 14px; background: var(--surface); border: 1px solid var(--line); border-radius: var(--radius); }
.toggle-row .name { font-weight: 500; }
.toggle-row .meta { font-size: 0.82rem; color: var(--ink-muted); }
.toggle-row.warn { border-color: var(--warn); }
.toggle-row.warn::after { content: "⚠ depends on a disabled item"; display: block; font-size: 0.8rem; color: var(--warn); grid-column: 1 / -1; }
.switch { position: relative; display: inline-block; width: 38px; height: 22px; }
.switch input { opacity: 0; width: 0; height: 0; }
.switch .slider { position: absolute; cursor: pointer; inset: 0; background: var(--line-strong); border-radius: 22px; transition: 0.2s; }
.switch .slider::before { position: absolute; content: ""; height: 18px; width: 18px; left: 2px; top: 2px; background: white; border-radius: 50%; transition: 0.2s; }
.switch input:checked + .slider { background: var(--accent); }
.switch input:checked + .slider::before { transform: translateX(16px); }
```

## Drag-and-drop JS skeleton (paste verbatim)

```html
<script>
  let dragId = null;
  document.querySelectorAll('.card-item').forEach(card => {
    card.draggable = true;
    card.addEventListener('dragstart', () => {
      dragId = card.dataset.id;
      card.classList.add('dragging');
    });
    card.addEventListener('dragend', () => card.classList.remove('dragging'));
  });
  document.querySelectorAll('.column').forEach(col => {
    col.addEventListener('dragover', e => { e.preventDefault(); col.classList.add('drop-target'); });
    col.addEventListener('dragleave', () => col.classList.remove('drop-target'));
    col.addEventListener('drop', e => {
      e.preventDefault();
      col.classList.remove('drop-target');
      const card = document.querySelector(`.card-item[data-id="${dragId}"]`);
      if (card) {
        col.appendChild(card);
        // update state: state.items[dragId].column = col.dataset.col;
      }
    });
  });
</script>
```

## Export pattern

```js
function buildExport() {
  // Walk the current state and produce the text the user will paste.
  // For a triage board: markdown grouped by column.
  // For toggles: a key=value block, or a JSON diff against initial state.
  // For an ordering: a numbered markdown list.
}

document.querySelector('.export-primary').addEventListener('click', async (e) => {
  await navigator.clipboard.writeText(buildExport());
  e.target.dataset.copied = '1';
  const label = e.target.textContent;
  e.target.textContent = 'Copied';
  setTimeout(() => { e.target.dataset.copied = ''; e.target.textContent = label; }, 1500);
});
```

## Cross-domain fit

- **Engineering ticket triage:** 30 issues drag across Now / Next / Later / Cut → export grouped markdown.
- **Conference talk shortlist:** 50 submissions drag across Accept / Maybe / Reject → export the accept list as a markdown table.
- **Wedding RSVP tracker:** 80 invitees toggle across Yes / No / Maybe / Dietary-note → export as CSV.
- **Feature-flag config:** 25 flags with dependencies, toggle on/off → export as the changed-only diff.
- **Reading list prioritization:** 40 books drag-to-reorder → export as numbered markdown list.

Same pattern, same export-first thinking, completely different domains.

## Anti-patterns

- **No export button.** The single most common way to get this pattern wrong. The artifact is half-built.
- **Export emits the whole page as HTML.** No — emit *the structured result.* Markdown, JSON, a diff, a config block. Something pastable into the *next* surface.
- **localStorage persistence by default.** State should live in memory. The user can refresh to start over; the export is how they keep work.
- **Faking interactions.** ("Click to cycle through statuses" when the user wanted drag-and-drop.) If you can't build the real interaction, switch patterns.
- **Empty initial state.** The user gave you data; load it. An empty editor is a generic tool, not a custom one.
- **Multiple primary CTAs.** One export button per page. If multiple exports are needed (markdown *and* JSON), one is primary and others are secondary text-buttons next to it.

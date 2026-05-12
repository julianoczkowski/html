---
name: table
description: Render tabular data as a sortable, filterable HTML table — click a column header to sort, type in the filter row to narrow, and export the current view as a markdown table or CSV. This skill should be used when the user has tabular data (incidents, customers, line-items, errors, leads, log rows, SKUs, candidates) with more than five rows or with multiple dimensions the reader will want to sort or filter by. Works for software (issue lists, log lines, perf metrics) and any domain (CRM rows, RSVP lists, inventory).
---

# table

Tabular data is the case where markdown is at its worst. A markdown table of 30 rows is unreadable; the reader can't sort, can't filter, can't search. An HTML table can. Same data, a different surface.

The artifact is the table itself plus the controls around it. The export emits the *currently visible, currently sorted* view — not the original input — so the user can paste the slice they actually wanted.

## Read first

Open `../templates/base.html` for tokens and `../gallery/interactions.html` for the filter / segmented-control / search primitives. The table sort and filter logic below is the pattern-specific addition.

## When the pattern fits

- The user has 5+ rows of structured data with 3+ columns.
- The reader will want to sort by at least one column ("most-impacted first", "oldest first", "by status").
- The reader will want to filter or search at least sometimes ("just the open ones", "just team A's").
- The result is a *slice* the user wants to paste somewhere — a planning doc, a Slack message, a spreadsheet.

## When it does NOT fit

- Fewer than 5 rows. Use a static markdown table or a `card` grid.
- The "rows" are not really tabular — they're long-form entries with paragraphs. Use `options` or `explainer`.
- The user wants to *edit* the table, not just read it. Use `editor` (and emit a table from its export).

## Structure

```
<kicker>Table · <topic>
<h1><What this table shows>
<prompt block>

<glance row>             ← optional: row count, last updated, hot stat
<controls bar>           ← search input + segmented-control filter + reset link
<table>                  ← sortable headers, sticky first row, monospace numeric cols
  <thead>                  click column → toggle sort asc/desc/none
  <tbody>                  rows hide via .hidden when filter excludes them
<sticky export bar>      ← "Copy as markdown" + "Copy as CSV"
```

## Rules

1. **Data lives in a single JS array, not in the HTML.** Source rows go into `const rows = [{...}, ...]` at the top of `<script>`. `render()` produces `<tr>`s from the array. This way sorting and filtering rebuild the DOM cheaply and exports walk the same array.
2. **One column is the natural sort.** Pick it; default-sort by it on load. (Most recent first, highest impact first, alphabetical by primary identifier.) Show the active sort with a `▲ / ▼` glyph in the header.
3. **Sortable columns get `data-sort="type"`** — `"text"`, `"num"`, `"date"`, `"status"`. The sort function uses the type to pick the comparator.
4. **Filter is text + categorical.** A search input that matches across all cells, plus a segmented control over the values of the most-used categorical column. Don't expose every filter possible — pick the one the reader actually wants.
5. **Numeric columns are right-aligned and monospace.** Counts, currency, durations. Use `text-align: right; font-family: "JetBrains Mono", monospace;`.
6. **Status columns use chips, not text.** Reuse `.chip.good / .chip.warn / .chip.bad / .chip.info` from `base.html`.
7. **Export emits the current view, not the original.** If the user sorted by impact and filtered to "open", `Copy as markdown` emits the open rows sorted by impact. The artifact is throwaway; the slice is the durable output.
8. **Two export buttons are fine when they're different formats.** Markdown for pasting into a doc, CSV for pasting into a spreadsheet. Markdown is primary; CSV is secondary.

## Pattern-specific CSS (extends base utility kit)

```css
.controls {
  display: flex; gap: 12px; align-items: center; flex-wrap: wrap;
  margin: 1.2em 0 0.6em;
}
.controls input[type="search"] {
  flex: 1; min-width: 220px;
  background: var(--canvas);
  border: 1px solid var(--hairline);
  border-radius: var(--r-md);
  padding: 10px 14px; font: inherit; font-family: "Inter", sans-serif;
}
.controls input:focus {
  outline: none;
  border-color: var(--coral);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--coral) 15%, transparent);
}
.controls .count {
  font-size: 0.82rem; color: var(--muted);
  font-family: "JetBrains Mono", monospace;
}

table.data {
  width: 100%; border-collapse: collapse; margin: 1em 0 2em;
  font-size: 0.92rem;
}
table.data thead th {
  position: sticky; top: 0;
  background: var(--canvas);
  border-bottom: 1px solid var(--hairline);
  text-align: left;
  font-family: "Inter", sans-serif;
  font-size: 0.72rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--muted);
  font-weight: 500;
  padding: 10px 12px;
  cursor: pointer;
  user-select: none;
}
table.data thead th[data-sort="num"]  { text-align: right; }
table.data thead th[data-sort="date"] { text-align: right; }
table.data thead th .arrow { color: var(--coral); margin-left: 4px; }
table.data tbody td {
  padding: 10px 12px;
  border-bottom: 1px solid var(--hairline-soft);
  color: var(--body);
  vertical-align: top;
}
table.data tbody td.num,
table.data tbody td.date {
  text-align: right;
  font-family: "JetBrains Mono", monospace;
  color: var(--ink);
}
table.data tbody tr:hover td { background: var(--surface-card); }
table.data tbody tr.hidden { display: none; }

.empty {
  padding: 2em 0;
  text-align: center;
  color: var(--muted);
  font-style: italic;
}
```

## Sort + filter JS skeleton (paste verbatim, adapt the rows + columns)

```html
<script>
  // 1. Data
  const rows = [
    { id: 1, title: "Login double-submit", team: "Auth",    status: "open",     impact: 8,  opened: "2026-04-12" },
    { id: 2, title: "Profile cache stale", team: "Profile", status: "open",     impact: 5,  opened: "2026-04-19" },
    { id: 3, title: "Captcha not local",   team: "Auth",    status: "closed",   impact: 3,  opened: "2026-03-28" },
    // ...
  ];

  // 2. State
  const state = { sort: { key: "impact", dir: "desc" }, q: "", filter: "all" };

  const STATUS_CHIP = { open: "warn", closed: "good", investigating: "info" };

  const cmp = {
    text:   (a, b) => String(a).localeCompare(String(b)),
    num:    (a, b) => Number(a) - Number(b),
    date:   (a, b) => new Date(a) - new Date(b),
    status: (a, b) => String(a).localeCompare(String(b)),
  };

  function visible() {
    let r = rows.slice();
    if (state.filter !== "all") r = r.filter(x => x.status === state.filter);
    if (state.q) {
      const q = state.q.toLowerCase();
      r = r.filter(x => Object.values(x).some(v => String(v).toLowerCase().includes(q)));
    }
    const { key, dir } = state.sort;
    const type = document.querySelector(`th[data-key="${key}"]`).dataset.sort;
    r.sort((a, b) => cmp[type](a[key], b[key]) * (dir === "asc" ? 1 : -1));
    return r;
  }

  function render() {
    const tbody = document.querySelector("table.data tbody");
    const r = visible();
    tbody.innerHTML = r.map(x => `
      <tr>
        <td>${x.title}</td>
        <td>${x.team}</td>
        <td><span class="chip ${STATUS_CHIP[x.status] || ''}">${x.status}</span></td>
        <td class="num">${x.impact}</td>
        <td class="date">${x.opened}</td>
      </tr>
    `).join("");
    document.getElementById("count").textContent = `${r.length} of ${rows.length}`;
    document.getElementById("empty").style.display = r.length ? "none" : "block";
    document.querySelectorAll("th .arrow").forEach(a => a.textContent = "");
    const active = document.querySelector(`th[data-key="${state.sort.key}"] .arrow`);
    if (active) active.textContent = state.sort.dir === "asc" ? "▲" : "▼";
  }

  // 3. Bindings
  document.querySelectorAll("th[data-key]").forEach(th => {
    th.addEventListener("click", () => {
      const k = th.dataset.key;
      if (state.sort.key === k) state.sort.dir = state.sort.dir === "asc" ? "desc" : "asc";
      else { state.sort.key = k; state.sort.dir = "asc"; }
      render();
    });
  });
  document.getElementById("q").addEventListener("input", e => { state.q = e.target.value; render(); });
  document.querySelectorAll(".segmented button").forEach(b => {
    b.addEventListener("click", () => {
      document.querySelectorAll(".segmented button").forEach(x => x.classList.remove("active"));
      b.classList.add("active");
      state.filter = b.dataset.v;
      render();
    });
  });

  render();
</script>
```

## Export pattern

```js
function buildMarkdown() {
  const r = visible();
  const headers = ["Title", "Team", "Status", "Impact", "Opened"];
  const sep = headers.map(() => "---").join(" | ");
  const lines = r.map(x => [x.title, x.team, x.status, x.impact, x.opened].join(" | "));
  return `| ${headers.join(" | ")} |\n| ${sep} |\n${lines.map(l => `| ${l} |`).join("\n")}`;
}

function buildCSV() {
  const r = visible();
  const headers = ["Title", "Team", "Status", "Impact", "Opened"];
  const esc = v => `"${String(v).replace(/"/g, '""')}"`;
  const lines = r.map(x => [x.title, x.team, x.status, x.impact, x.opened].map(esc).join(","));
  return [headers.map(esc).join(","), ...lines].join("\n");
}

document.getElementById("copyMd").addEventListener("click",  e => copy(e, buildMarkdown));
document.getElementById("copyCsv").addEventListener("click", e => copy(e, buildCSV));

async function copy(e, fn) {
  await navigator.clipboard.writeText(fn());
  const label = e.target.textContent;
  e.target.dataset.copied = "1";
  e.target.textContent = "Copied";
  setTimeout(() => { e.target.textContent = label; e.target.dataset.copied = ""; }, 1500);
}
```

## Cross-domain fit

- **Incident list:** rows = incidents, columns = title / service / status / severity / opened. Default sort by severity desc.
- **CRM accounts:** rows = accounts, columns = name / segment / status / ARR / last touch. Default sort by ARR desc.
- **RSVP roster:** rows = guests, columns = name / party-size / yes-no-maybe / dietary / table. Default sort by status.
- **Slow queries:** rows = query hashes, columns = first line / db / mean ms / p95 / call count. Default sort by p95 desc.
- **Candidate pipeline:** rows = candidates, columns = name / role / stage / referred-by / applied. Default sort by stage.

## Anti-patterns

- **HTML rendered server-side from a static row list.** Then sorting and filtering don't work. Always render `<tbody>` from a JS array.
- **Sort without an active-indicator glyph.** The reader has to click to discover the current sort. Show `▲` / `▼` on the active column.
- **Filter chips for every column.** Pick the *one* categorical column the reader actually slices by. Other filters belong in the search input.
- **Export emits the original input.** The whole point is to emit the current view — the slice the user just built.
- **No export.** Same trap as `editor` — the artifact is half-built. Add at least one export button.
- **Numeric columns left-aligned in proportional type.** Eye can't compare values. Right-align, monospace.

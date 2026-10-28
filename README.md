# Ads Studio

A Google Ads console rebuilt in plain English and Apple's design language.

One HTML file. No build step, no dependencies, no network calls. All data is example
data for a fictional demo account (Harbor & Pine Dental) and lives in JS objects at
the top of the `<script>` block.

## Running it

Serve it over HTTP rather than opening the file directly:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

`file://` works for a quick look, but browsers treat it as an opaque origin, so
`localStorage` throws and the app silently stops remembering your date range,
wording preference and dismissed recommendations. Everything still renders — the
storage calls are wrapped in try/catch — but you cannot test persistence that way.

## What it does

| Page | What it is |
| --- | --- |
| Today | Month-to-date budget pacing, a prioritised fix list, four KPIs, a day-by-day chart |
| Campaigns | Inline budget editing, on/off switches, search, a detail panel |
| What people typed | The search terms report, sortable, with Quality Score as "Ad health" |
| New campaign | A five-question builder with a live SERP preview and a forecast |
| Your ad | Responsive search ad editor with character counts, Ad Strength, sitelinks and callouts |
| Google's words | The jargon-to-plain-English glossary |

The sidebar switch flips every label in the app between plain English and Google's
own terminology. Both are kept in `data-plain` / `data-jargon` attributes or the
`W()` helper, so adding a new label means supplying both.

## How it is put together

Single file, in this order:

1. `:root` design tokens — colour, type, radii, easing. Apple's system palette, with
   a full dark theme defined three ways (bare `:root`, `prefers-color-scheme`, and
   `[data-theme]`) so it is correct whether the viewer picked a theme or left it on
   system.
2. Component CSS, then responsive rules at 1150 / 1120 / 900 / 760 / 620 / 440px.
3. Markup — one `<section class="page">` per view, toggled with `hidden`.
4. Script — example data, helpers, then one `render*()` function per surface.

Conventions worth keeping:

- **No framework and no CDN.** Keep it that way unless there is a real reason.
- **`buildSeries()` is the single source of truth** for everything on Today. It
  generates twice the selected window so the first half is the previous period and
  the second half is on screen. The chart bars, every sparkline, each KPI value and
  every percentage delta read off those same buckets. Do not reintroduce hardcoded
  deltas.
- **Undo over confirmation.** Destructive actions snapshot state as JSON and pass a
  restore closure to `toast()`. The whole mutable state is small enough that this is
  cheaper than writing an inverse per action.
- **`store()` swallows storage errors** and returns null. Every read must cope with
  null.
- **Real numbers only.** If a figure is shown, it should be derived. The point of the
  app is that Google Ads hides the one number that matters (cost per booked patient)
  behind forty that do not.

## Known limitations

- Campaign and keyword edits are in-memory. Only the date range, wording preference
  and dismissed recommendations persist; a reload resets the rest by design.
- The forecast in the campaign builder is a plausible model, not a real Keyword
  Planner query. Same for the daily series.
- `prompt()`, `confirm()` and `window.print()` do nothing inside a published
  Claude artifact, so the UI never relies on them.

## Testing

There is no test runner. Verification has been done by driving the page in a browser
and asserting against the DOM. The checks worth repeating after a change:

- No page-level horizontal scroll on any view at any width (a wide table may scroll
  inside its own `.scroll-x` container).
- No `NaN`, `undefined` or `[object Object]` in visible text across all views and all
  three date ranges.
- Both empty extremes: every campaign paused (the monthly ceiling becomes zero) and
  every keyword blocked.
- Junk in the budget field: `abc`, `-5`, `0`, `999999`, `$60`, `1,200`, `12.7`, empty.
- Undo restores after a pause, a budget edit, a keyword block and an applied fix.

## Origin

Built with Claude Code. Also published as a Claude artifact at
<https://claude.ai/artifact/S5va1pu2tkGPn4B9tyFVSd> (private to the owner).

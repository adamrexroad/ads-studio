# Working on Ads Studio

Read `README.md` first — it covers the structure, the conventions and the manual
test checklist. This file is the short version for a coding session.

## Ground rules

- **One file.** Everything lives in `index.html`. No build step, no package manager,
  no CDN scripts. If a change seems to need a dependency, say so before adding one.
- **Serve over HTTP to test** (`python3 -m http.server 8000`). On `file://` the
  origin is opaque, `localStorage` throws, and persistence cannot be exercised.
- **Verify in a browser before claiming a fix.** This project has no test runner, so
  "it should work" is not evidence. Drive the page and assert against the DOM.

## Things that have already bitten

- `parseFloat(x) || fallback` silently swallows `0`. The budget field used to ignore
  a typed zero because of this.
- Stripping non-digits with a regex turns `-5` into `5`. Validate the raw string.
- A percentage of a zero total is `NaN`; assigning `"NaN%"` to `style.width` is
  rejected, so the element keeps its *previous* value and the bug looks like stale
  state. Guard the zero case explicitly.
- Re-seeding a list inside a render function makes it impossible to delete the last
  item. Seed once at startup.
- `::after` on an `<input>` is not reliably painted. The switches use a sibling
  `<span class="sw-track">` instead.
- `getComputedStyle` during a CSS transition returns the in-flight value, not the
  target. Disable transitions or wait before asserting on one.
- Re-rendering replaces DOM nodes, so a cached element reference goes stale. Re-query
  after any render when testing.

## Where things are

- Example data: `campaigns`, `keywords`, `FIXES`, `GLOSSARY` near the top of the
  script.
- Everything on the Today page derives from `buildSeries()`. Change the data there,
  not in the individual renderers.
- One `render*()` function per surface; `renderAll()` orchestrates. `go(id)` switches
  views and is also the deep-link handler via `location.hash`.
- Plain-English and Google wording are paired everywhere — `data-plain` /
  `data-jargon` attributes, or the `W(plain, jargon)` helper. A new label needs both.

## Tone of the copy

Plain, specific, and about money. Name things the way the person running the
practice would ("new patients", "cost per patient"), not the way the ad platform
does. The glossary page carries the translation. Avoid exclamation marks, hedging
and marketing voice.

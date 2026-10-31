# Ads Studio

A simpler, more usable Google Ads console. It reads account performance and
turns it into specific suggested decisions, each one priced in money per month,
instead of leaving the operator to work them out from forty separate metrics.

> **This is baseline architecture and layout for much larger client software
> builds.** It is the starting point those builds are grown from, not a finished
> product.
>
> **The ad strategies and the analytics that produce the suggestions are not in
> this repository.** What is here is the structure that presents them and the
> model for how a decision is stated. The logic that decides which decisions are
> worth surfacing belongs to a client build.

![The Today page](docs/screenshots/today.jpg)

## What this repository is

Baseline architecture and layout. The derived-metrics model, the decision layer,
the two-vocabulary label system and the view structure here are the foundation
that much larger client software gets built on top of. What is in this repository
is the layer that does not change from one build to the next.

A client build keeps this core and adds what that account needs: a real Google
Ads API connection behind the data, OAuth and account selection, write-back for
budget and status changes, conversion imports from the phone system or the
booking form, scheduled reporting, and the strategy and analytics engine that
decides what to recommend.

Read it as a structural reference, not as a product.

## What it does

It answers the question a Google Ads account does not answer directly, which is
what a booked customer costs and whether that is worth paying.

The interface reports roughly forty numbers. Impressions, click-through rate,
average position, Quality Score, conversion rate, cost per click: all real, none
of them the one that decides whether to keep spending. That one is cost per
booked customer set against what a customer is worth, and it has to be assembled
by hand.

Ads Studio leads with it. Spend is shown paired with what the spend bought. Every
suggested decision carries a dollar figure per month rather than a severity
score. A switch flips every label in the application between everyday wording and
Google's own vocabulary, so an operator can read the two against each other
rather than having to learn one before starting.

### Suggested decisions

The list on the Today page is the working surface. Each entry states the finding,
the reason behind it, the money involved, and a single button that performs it:

> **"cheap teeth whitening" has burned $361 for one patient.** 188 clicks, 1
> booking. The people searching it are price shopping, not booking. Turning it
> off keeps the rest of that campaign running.
> *Frees about $340 a month.*

What produces those findings is a strategy and analytics layer that is **not in
this repository**. The entries here are modelled, and they exist to establish the
shape: a finding, a reason, a number, one action, and the ability to dismiss it.

## The six views

![Campaigns](docs/screenshots/campaigns.jpg)

| View | What it does |
| --- | --- |
| Today | Month-to-date pacing, the suggested decision list, four KPIs, a day-by-day chart |
| Campaigns | Inline budget editing, on and off switches, search, a detail panel |
| What people typed | The search terms report, sortable, with Quality Score renamed "Ad health" |
| New campaign | A five-question builder with a live search-result preview and a forecast |
| Your ad | Responsive search ad editor with character counts, Ad Strength, sitelinks and callouts |
| Google's words | The glossary that maps one vocabulary onto the other |

The search terms report is where waste becomes visible. A term with 188 clicks
and one booking sits next to one with 331 clicks at 42 cents, and the column that
decides anything is the rightmost.

![What people typed](docs/screenshots/search-terms.jpg)

The campaign builder asks five questions in order and shows the ad and the
forecast updating beside them, so the budget slider and the share of searches you
would win move together.

![New campaign](docs/screenshots/new-campaign.jpg)

The ad editor enforces Google's real limits, 30 characters a headline and 90 a
description, and scores Ad Strength the way Google broadly does, which is mostly
a count of how many distinct headlines it has to test with.

![Your ad](docs/screenshots/your-ad.jpg)

## Architecture

One HTML file. No build step, no package manager, no framework, no CDN script and
no network call. The entire application is `index.html`: design tokens, component
CSS, markup for six views, then the script.

### Everything on Today derives from one series

`buildSeries()` is the single source of truth. It generates twice the selected
window, so the first half is the previous period and the second half is what is
on screen. The chart bars, every sparkline, each KPI value and every percentage
delta read off those same buckets.

This is why no figure in the app can disagree with another one. There are no
hardcoded deltas anywhere, and reintroducing one would be the way to break it.

### Every label exists in two vocabularies

Labels are paired, either as `data-plain` and `data-jargon` attributes or through
the `W(plain, jargon)` helper. The switch flips all of them at once and the
choice persists.

A new label needs both halves. That constraint is the feature: it forces a
decision about what a thing is actually called, rather than letting the platform's
name for it leak through by default.

### Decisions are priced, not scored

Every entry in the decision list carries a figure in money per month, "Frees about
$340 a month" rather than a severity badge, and exactly one action. This is the
part a client build plugs its own analytics into. The presentation contract is
fixed here; what fills it is not.

### Undo rather than confirm

Destructive actions snapshot the whole mutable state as JSON and pass a restore
closure to the toast. The state is small enough that this is cheaper than writing
an inverse for every action, and it means nothing in the app interrupts you to ask
whether you meant it.

### Themes and widths

![Dark mode](docs/screenshots/today-dark.jpg)

The dark theme is declared three ways: on bare `:root`, under
`prefers-color-scheme`, and under an explicit `[data-theme]`. All three exist so
the app is correct whether the viewer chose a theme or left it on system.

Responsive rules sit at 1150, 1120, 900, 760, 620 and 440 pixels. The campaign
table drops columns rather than scrolling the page, and the sidebar becomes a
drawer.

## Running it

Serve it over HTTP rather than opening the file directly:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

`file://` renders fine but browsers treat it as an opaque origin, so
`localStorage` throws and the app silently stops remembering your date range,
wording preference and dismissed decisions. The storage calls are wrapped in
try/catch, so nothing breaks visibly, which is exactly why it is worth knowing.

## Honest status

This is a working build, not a shipped product. The gaps below are the ones a
client build closes.

- **The strategy and analytics layer is not here.** The suggested decisions are
  modelled examples that establish the format. Nothing in this repository decides
  what is worth recommending, scores an opportunity, or analyses an account. That
  engine is the part a client build supplies.
- **It does not connect to Google Ads.** There is no API client, no OAuth and no
  account linking anywhere in this repository. Every figure is generated locally.
- **The data is fictional.** Harbor & Pine Dental does not exist, the domain in
  the previews does not resolve, and the account number is invented. No real
  advertiser, spend figure or client of any kind is in this repository.
- **The forecast is a model, not Keyword Planner.** The campaign builder's
  projected clicks, cost per click and share of searches are arithmetic on the
  budget and the goal. They are not a query against Google's volume data and they
  will not match it.
- **Campaign and keyword edits are in-memory.** Only the date range, the wording
  preference and dismissed decisions persist. A reload resets the rest, by design.
- **No tests.** Verification has been done by driving the page in a browser and
  asserting against the DOM: no page-level horizontal scroll at any width, no
  `NaN` or `undefined` in any visible text across all six views and all three date
  ranges, both empty extremes (every campaign paused, which takes the monthly
  ceiling to zero, and every keyword blocked), and junk in the budget field.
- **Ad Strength is an approximation.** It counts distinct headlines and
  descriptions the way Google broadly does, without Google's model behind it.

## Licence

No licence is granted. This repository is published so the architecture can be
read, not reused.

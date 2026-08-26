# IDV Production Dashboard — Mockup Spec

This document describes a working HTML/CSS/JS mockup of a “Production Dashboard” that shows factory production output either in units or in Item Difficulty Value (IDV) — a per-item difficulty/complexity score. It is a single self-contained page (sidebar + top bar + one dashboard view).

The goal of this document is to describe everything the mockup does — layout, every button/control, the table’s structure, and how things behave — in enough detail that someone who has never seen the mockup could rebuild it from scratch, in a different tech stack, without needing to open the original file.

## 1. What “Item Difficulty Value (IDV)” means, briefly

Every manufactured item (a jersey, a sock, a jacket, etc.) has a fixed IDV score — a number like 1.5 or 4.2 — representing how difficult/time-consuming that item is to produce. Higher = harder. The dashboard can show raw unit counts (how many items were made) or the same data expressed as total IDV (units × each item’s IDV, summed), which better reflects production effort than a raw unit count does — e.g. 1,000 easy socks vs. 1,000 hard jackets look identical as “1,000 units” but very different in IDV terms.

Each item also belongs to:
- a factory (where it’s made),
- one or more brands (some items are shared across multiple brands),
- a sport/category (e.g. Football, Volleyball, Apparel).

## 2. Overall page layout

The page is a classic app shell: a left sidebar, a top bar, a breadcrumb sub-header, and a scrollable content area. Top to bottom / left to right:

```
┌───────────┬─────────────────────────────────────────────┐
│           │  [≡] [QUICKSTRIKE ▾]                    [🔔] │  ← Top bar
│  Sidebar  ├─────────────────────────────────────────────┤
│           │  Home › Dashboard › Production Dashboard     │  ← Breadcrumb
│           ├─────────────────────────────────────────────┤
│           │  Year-over-Year Snapshot                     │
│           │  [card][card][card][card][card][card][card]  │  ← KPI cards
│           │                                               │
│           │  ┌─────────────────────────────────────────┐ │
│           │  │ Unit output   [2025 vs 2026]             │ │
│           │  │ (Units) (Item Difficulty Value)  Years▾ Factories▾│ ← Toolbar
│           │  ├─────────────────────────────────────────┤ │
│           │  │           [ the big pivot table ]        │ │  ← Main table
│           │  └─────────────────────────────────────────┘ │
└───────────┴─────────────────────────────────────────────┘
```

### 2.1 Sidebar (left)

- User card at the top: a circular user-icon avatar, the user’s name (“Seo Administrator”), their role (“Admin”), and a 3-line “menu” icon on the far right of the row (visual only in the mockup — no menu wired up). There is no logo above it — that was removed from the current mockup.
- Nav section 1 — “Analytics Reports” (section label in small caps):
  - “Dashboard” nav item, with a small bar-chart icon and a rounded badge showing the number 5 on the right edge (visual only — represents “5 sub-pages”).
  - Underneath it, a list of sub-items (indented, smaller text): Overview, Old Overview, Production, Sales, Sales in Units, and Production Dashboard — this last one is the current page and is visually highlighted (bold, dark text) to show it’s active.
- Nav section 2 — “Acquisition”:
  - “Engagements” nav item with a people icon and a badge showing 1.
  - One sub-item underneath: “Pages and Screens”.
- Nothing else below that — the rest of the sidebar is empty space.
- On narrow/mobile screens: the sidebar becomes a slide-out drawer. It’s hidden off-screen by default and slides in from the left when the hamburger button (top-left, see below) is tapped. A dark semi-transparent backdrop appears behind it; tapping the backdrop closes the drawer again.

### 2.2 Top bar

Height ~48px, white background, thin bottom border. Contains, left to right:

- A hamburger/menu icon button (3 horizontal lines) — its job is to toggle the mobile sidebar drawer open/closed. (On desktop it’s still present and clickable, it’s just not needed since the sidebar is already visible.)
- A thin vertical divider.
- A brand switcher dropdown, styled like a small clickable pill: a small colored icon square, the text “QUICKSTRIKE”, and a chevron arrow. Clicking it opens a dropdown panel containing:
  - A search box at the top (“Search brand…”) — typing filters the list below in real time (case-insensitive, matches anywhere in the brand name).
  - A first entry, “QUICKSTRIKE — All Brands”, which is the default/selected option — represents “no brand filter, show everything”.
  - Below that, one row per individual brand (23 brands total in the current mockup, e.g. Riddell, New Balance, Baden, Richardson, Sport-Tek, etc.) — each row shows a short brand code plus its full name (e.g. “RDL — Riddell”).
  - Only one option can be selected at a time (selecting a specific brand deselects “All Brands”, and vice versa). The dropdown closes immediately after a selection is made. The button’s label updates to show either “QUICKSTRIKE” (all brands) or the chosen brand’s name.
  - Selecting a specific brand filters the whole dashboard to only that brand’s share of production — this affects both the KPI cards and the main table (see section 5 for exactly how).
- On the far right: a notification bell icon with a small red dot badge in its corner (visual only in the mockup — not wired to open anything).

### 2.3 Breadcrumb / sub-header

A slim row just below the top bar showing: Home › Dashboard › Production Dashboard (the last segment, the current page name, is bold/dark; the rest is muted gray). This row also has room on the right for an optional date-range indicator, but that’s commented out / unused in the current mockup — not something to implement.

## 3. KPI cards (“Year-over-Year Snapshot”)

Directly above the main table sits a header line reading “Year-over-Year Snapshot” with a smaller gray sub-label next to it: “Current year vs. last year”. This label is deliberately plain-language and doesn’t spell out a specific month range or the word “(fixed)” — see the note below for what it actually means under the hood.

Below that header is a row of cards, one per currently-visible factory (the row of cards automatically has as many columns as there are visible factories, up to 7 wide before it would need to wrap). Each card shows:

- The factory’s short code at the top (e.g. “BLB”, “HCM”), colored using that factory’s assigned color (see section 7).
- A large bold number — the factory’s current-period total.
- Below that, a smaller line showing the percentage change vs. the prior year, with an up or down arrow, colored green if the change is positive/flat, red if negative (e.g. “↑ 12%”). Note this is just the percentage — it does **not** append “vs 2025” or any other year label next to it.

**Important behavior:** this comparison window is fixed, not affected by the Year/Factory filters below the table — but internally it is *not* hardcoded to a specific month range. The cards always compare the current year’s year-to-date totals (January through whichever month is the most recent one with real data) against the prior year’s totals for that same span of months. This window automatically advances as later months of the current year get real data — e.g. it covers Jan–Aug once August data exists, and will cover Jan–Sep the moment September data lands, with no code change required. The visible label doesn’t spell any of this out to the user (it just says “Current year vs. last year”), but the underlying month range must still be computed dynamically, not hardcoded to a specific month — that dynamic behavior is what never changes, regardless of the Year filter below: the cards always answer “how are we doing this year vs. last year so far,” for whatever “so far” currently means.

**However, the metric shown in each card must switch with the main Units / Item Difficulty Value toggle** (section 4.1) — i.e. only the time window and the “current vs. prior year” comparison basis are fixed; the number itself (unit count or IDV total) tracks whichever view is currently active, the same way the main table does. Concretely: the same card shows one number in Units mode and a different number in IDV mode — only the year-to-date / current-vs-prior-year comparison basis never moves.

The KPI row does, however, update its set of cards (which factories appear) when the Factory filter changes, and its brand-filtered totals when the brand switcher (section 2.2) changes — it just never changes its current-vs-prior-year time window, whatever that window's current month range happens to be.

## 4. Toolbar (view toggle + filters)

This sits directly above the main table, inside the same card/panel as the table. It has two rows on the left and a filter block on the right (or below, depending on width):

### 4.1 Left side — title, badge, and view toggle

- A title, “Unit output” (or “Item Difficulty Value output” when the IDV view is active — see 4.3) — updates automatically to match the second row’s toggle state.
- A rounded badge next to the title showing the current year selection, e.g. “2025 vs 2026” if two years are selected, or just “2026” if only one year is selected.
- Below that: a segmented two-button toggle, “Units” / “Item Difficulty Value”. Exactly one is active at a time, shown with a white pill background against a gray track (like an iOS-style segmented control). “Units” is the default/starting selection.

### 4.2 Right side — filters

Two filter rows, each with a label and a dropdown button:

- **Years filter** — label “Years”, a dropdown button showing either the selected years (e.g. “2025, 2026”) or “All years” if every year is selected. Clicking it opens a panel with:
  - A search box (“Search year…”) to filter the list by typing digits.
  - An italic “— Any —” option at the top that selects all years at once.
  - One row per year, 2021 through 2026, each with a checkmark shown when selected. Multiple years can be selected simultaneously (multi-select, not single-select).
  - At least one year must always remain selected — clicking to deselect the very last remaining checked year is a no-op (nothing happens); the user must select something else first if they want to fully replace their selection.
  - Default selection on page load: the two most recent years (2025 and 2026).
- **Factories filter** — label “Factories”, same dropdown pattern as Years: search box, an “— Any —” option that selects every factory, checkboxes per factory (each checkbox rendered in that factory’s own color when checked), and the same “at least one must stay selected” rule. The button label shows either “All factories” or a comma-separated list of the selected factory codes (e.g. “BLB, HCM, PMP”). Default: all factories selected.

Both dropdowns share the same interaction pattern: click the button to open/close it, click anywhere outside the dropdown to close it, and typing in the search box narrows the visible rows without affecting what’s actually selected.

### 4.3 Effect of the view toggle

Switching between “Units” and “Item Difficulty Value” doesn’t change what filters are available — it changes what number is shown in every cell of the table **and in every KPI card** (see section 3 — the KPI cards’ time window stays fixed, but the metric they display tracks this toggle). In “Units” mode, cells show a straightforward unit count. In “IDV” mode, each cell shows the total IDV for that same slice of data — i.e., for every item that would have contributed to that unit count, take (that item’s share of the units) × (that item’s IDV score), and sum it up. This means switching to IDV view can make two cells with identical unit counts show very different numbers, if the mix of items differs.

## 5. Main table (the pivot table)

This is the centerpiece of the dashboard and the most structurally complex part. It is not a simple list of rows — it’s a pivot/cross-tab table.

### 5.1 What the rows and columns represent

- Rows: one per calendar month (January through December), always in that fixed order, plus one extra “Totals” row at the very bottom that sums the whole column above it. The Totals row is visually distinct (shaded background, bold text).
- Columns: for every factory that’s currently checked in the Factories filter, there are two groups of columns — “Approved” and “Shipped” — and within each group, one column per currently-selected year. So if 3 factories and 2 years are selected, that’s 3 factories × 2 metrics × 2 years = 12 data columns, plus (if more than one factory is visible) an extra “Totals” column group (also split into Approved/Shipped × years) that sums across all visible factories.
- If only one factory is selected, the “Totals” column group is hidden entirely (it would be redundant — same numbers as the single factory).

### 5.2 The header — three stacked rows

1. **Top header row:** one wide cell per factory, spanning all of that factory’s columns, showing the factory’s full name (not just the code) — e.g. “BLB - Billerby Corporation”. Each factory’s header cell is tinted with a soft background color unique to that factory (see section 7), with a slightly stronger-colored line along its bottom edge. This header cell is **not** clickable and has no icon in it — it’s purely a label. (An earlier version of the mockup had a small icon here, then briefly made the whole cell clickable to open a factory-wide item catalog; both were removed. The only way into the item-breakdown modal now is by clicking an actual data cell — see 5.3 and section 6.) If a Totals column group is showing, it gets its own header cell here too, styled in a dark neutral color (deliberately different from any factory’s color, so it can’t be mistaken for “just another factory”); the Totals header cell is also not clickable.
2. **Second header row:** under each factory’s name, two narrower cells labeled “Approved” and “Shipped”, each spanning that group’s year columns.
3. **Third header row:** under each Approved/Shipped group, one narrow cell per selected year (e.g. “2025”, “2026”). The most recent selected year in each group is visually emphasized (bold, colored background matching the factory) to draw the eye to “this year” vs. comparison years.

The very first column (leftmost, under all three header rows) is just labeled “Month” (visually blank in the header, but that’s what it is) and contains the month names down the left side.

### 5.3 Data cells

- Every number is formatted with thousands separators (e.g. “12,345”), and a zero or empty value is shown as an em-dash “—” rather than “0”.
- The most recent selected year’s column in each Approved/Shipped group is styled darker/bolder than older years, to distinguish “current” from “comparison” data at a glance.
- Any cell with a non-zero value is clickable — hovering it shows an underline and a “View item breakdown” tooltip. Clicking it opens the item-breakdown modal (section 6), scoped to exactly that factory + month + metric (Approved or Shipped) + year.
- Cells representing a future period with no data yet (in the mockup: months from May 2026 onward, since data only exists through April 2026) are shown as “—” and are not clickable.
- The bottom Totals row is clickable per factory, the same way a regular month cell is — clicking a given factory’s cell in that row opens the modal in “full year” mode instead of “single month” mode (see 6.1).
- The right-hand Totals **column group** (the one that sums across every visible factory, shown both as extra columns in each month row and as the rightmost block of the bottom Totals row) is **not** clickable, in either place. That’s deliberate, not an oversight: it doesn’t correspond to any single factory’s item catalog, so there’s nothing for the modal to scope a breakdown to. Only cells that resolve to one specific factory (any month cell, or a factory’s own cell in the bottom Totals row) open the modal.

### 5.4 Sticky/pinned columns and rows

- The leftmost “Month” column stays pinned in place (doesn’t scroll away) when the table is scrolled horizontally — so you can always tell which row is which even after scrolling right to see later factories/years.
- If a Totals column group is visible, all of its columns stay pinned to the right edge of the table while scrolling horizontally, for the same reason — you can always see the running totals no matter how far left/right you’ve scrolled through the individual factories.
- Both of these pinned regions need their exact pixel widths recalculated any time the table’s content changes (e.g. filters change how many columns exist), so the pinning stays aligned with the columns underneath.

### 5.5 Table on narrow/mobile screens

The table always stays a real table — it does not collapse into stacked cards or anything like that on small screens. Instead, on screens narrower than roughly 768–900px, the dashboard defaults to showing only one year and one factory at load time (so the resulting table is narrow enough to read without much horizontal scrolling). This is only the starting selection — the Year and Factory dropdowns are still fully interactive, so a mobile user can select more if they choose to; it’s just a friendlier default. Below roughly 480px, some text (logo, header, KPI values) also shrinks slightly to fit.

## 6. Item breakdown modal (popup)

Triggered only by clicking a clickable data cell in the table, as described in 5.3 (a month cell, or a factory’s own cell in the bottom Totals row) — the table header is not a trigger, and neither is the aggregate Totals column group (see 5.2 and 5.3).

### 6.1 The one mode this modal has

There used to be a second “factory-wide catalog” mode (triggered from the table header, showing every item a factory produces regardless of period) — it’s been removed. The modal now only has one mode:

- **Period-scoped breakdown** (triggered by clicking a month cell, or a factory’s own cell in the bottom Totals row): shows only the items that make up that specific month (or that factory’s full year, if its Totals-row cell was clicked) for that factory and metric (Approved or Shipped). The quantities shown for each item reflect that specific period’s total, split across items proportionally to their typical share (weighted by each item’s known order quantity), and rounded so the numbers still add up exactly to the period’s real total.

### 6.2 Modal header

- Title: the factory’s full name plus the specific period — e.g. “BLB - Billerby Corporation — March 2026” or “BLB - Billerby Corporation — 2026 — Full Year”.
- Subtitle: a one-line summary, “Approved (or Shipped) breakdown — N items — X,XXX units”. That last figure tracks the main Units/IDV toggle just like everything else: in Units view it reads “X,XXX units” (the raw quantity total for that period); in IDV view it instead reads “X,XXX.X total IDV” (the summed IDV total for the same period) — the item count (“N items”) doesn’t change between the two, only the trailing total does.
- A thin colored accent bar directly under the header, colored to match the factory’s assigned color.
- A small “×” close button in the top-right corner.

### 6.3 Modal body — the item table

Columns, left to right:

1. **Item name** — the item’s full descriptive name, can wrap to multiple lines if long (e.g. “Compression Pant, Full Length, Men and Youth”). Note: the actual column header text in the current mockup literally reads **“Master Block Name”** — a legacy label carried over on purpose from the old system rather than a generic “Item” header. Rebuild it with that exact header text, not a paraphrase, if matching the current mockup is the goal.
2. **Sport** (e.g. “Football”, “Volleyball” — shown as the full sport name, not an abbreviation; an item that applies to more than one sport lists all of them, comma-separated).
3. **Brand** (the brand code(s), e.g. “PLS” or “RDL”. An item made for more than one brand must list every brand it belongs to, comma-separated — not just one. This matters because some items are shared programs across brands and it would be misleading to attribute them to only a single brand.)
4. **IDV** — the item’s difficulty score (e.g. “2.25”), shown as a colored number. The color runs along a green → yellow-green → yellow → orange → red gradient as the IDV score increases from low (easy) to high (hard) — this is a quick visual cue for “how difficult is this item” without having to read the number itself.
5. **Quantity** — how many units of this item are represented: its computed share of the clicked period’s total (see 6.1).
6. **IDV Total** — Quantity × IDV for that row.
7. **%** — what percentage of this modal’s overall total IDV this one row represents (so the rows can be scanned to see which items are driving the bulk of the difficulty/effort, not just the bulk of the unit count).

Rows highlight on hover. If there are more rows than fit in the visible area, only the table body scrolls — the column headers stay pinned at the top of the modal while scrolling.

### 6.4 Closing the modal

Three ways to close it in the mockup:

1. Clicking the “×” button in the header.
2. Pressing the Escape key.
3. Clicking anywhere on the dark overlay outside the modal box (i.e. click-outside-to-close). This is a deliberate mockup behavior worth flagging explicitly to whoever rebuilds this: some component libraries default to not closing a modal on backdrop click (to avoid accidental data loss on forms) — since this modal is purely read-only/informational (no form, nothing to lose), click-outside-to-close is safe and expected here, and rebuilding it without that behavior would be a regression from the mockup’s actual behavior, not a neutral stylistic choice.

## 7. Colors and visual identity per factory

Each factory has a small fixed palette assigned to it — a main color, a darker text-safe variant of that color, a light background tint, and a border color — used consistently across the KPI card label, the factory’s checkbox in the Factories filter, the factory’s table header cells, and the accent bar at the top of its item-breakdown modal. Every factory has a visually distinct color so they stay identifiable at a glance across every part of the dashboard (12 factories currently: distinct oranges, pinks, blues, purples, greens, etc. — no two the same hue). The Totals grouping (the table’s Totals column group) uses a separate dark neutral color on purpose, so it never gets confused with an actual factory’s color.

A handful of factories (5 in the current mockup: CPA, JCG, MDJ, QTS, SPY) exist in the system but have no production data yet — they still appear normally in the Factory filter and would render normally in the table (all zeros / dashes) if selected, since they represent factories that have been added to the roster but haven’t started shipping orders.

## 8. Summary checklist of interactive elements

For a quick reference, every clickable/interactive control on this page:

- [ ] Hamburger button (top-left) — toggles mobile sidebar drawer
- [ ] Mobile sidebar backdrop — click to close drawer
- [ ] Brand switcher dropdown (top bar) — single-select, with search
- [ ] Notification bell — visual only, not wired to anything
- [ ] Sidebar nav items/sub-items — visual only in this mockup (no routing wired up beyond marking “Production Dashboard” as the active page)
- [ ] Units / Item Difficulty Value segmented toggle — switches the table's numbers + the toolbar title, **and also switches the number shown in every KPI card** (only the KPI cards' current-vs-prior-year comparison *basis* stays fixed — the month range itself is dynamic and advances as new months of data land — see section 3)
- [ ] Years filter dropdown — multi-select, search, “Any” = select all, minimum one must stay selected
- [ ] Factories filter dropdown — multi-select, search, “Any” = select all, minimum one must stay selected, checkbox colored per factory
- [ ] Any non-blank data cell that resolves to one specific factory — a month cell for that factory, or that factory’s own cell in the bottom Totals row — opens the item-breakdown modal for that specific period. The aggregate Totals **column group** (summed across all visible factories) is not clickable anywhere it appears, and neither is the factory header cell — both are plain, non-interactive labels.
- [ ] Modal “×” close button
- [ ] Modal Escape key
- [ ] Modal click-outside-to-close (see section 6.4)

## 9. Data & API expectations (assumed — for illustration only)

The mockup currently runs entirely on hardcoded sample data baked into the page itself. None of this section is a confirmed backend design — it’s a set of reasonable, industry-standard placeholder contracts, shaped directly around the mockup’s existing sample data, so a frontend rebuild has something concrete to code against instead of guessing on every payload shape while the real API is still being designed. Treat every endpoint name, field name, and response envelope below as a starting proposal, not a spec to be built against as-is without backend sign-off.

General conventions assumed throughout: JSON over HTTPS, a `{ "data": ..., "meta": ... }` response envelope, camelCase field names, ISO-formatted dates/years as integers where whole years are meant, monetary/quantity values as plain numbers (not strings), and standard HTTP status codes for errors (with an `{ "error": { "code", "message" } }` body on failure).

### 9.1 Reference/lookup data (rarely changes — safe to cache client-side)

**`GET /api/factories`** — the factory roster, including factories with no production data yet (see section 7). Drives the Factories filter, the table’s column groups, and each factory’s assigned display color.

```json
{
  "data": [
    {
      "id": "blb",
      "code": "BLB",
      "fullName": "BLB - Billerby Corporation",
      "color": "#EF9F27",
      "textColor": "#854F0B",
      "backgroundColor": "#FFF4EC",
      "borderColor": "#EF9F27",
      "hasProductionData": true
    },
    {
      "id": "cpa",
      "code": "CPA",
      "fullName": "CPA - Cap America",
      "color": "#D4527E",
      "textColor": "#7A2A48",
      "backgroundColor": "#FCE9F0",
      "borderColor": "#D4527E",
      "hasProductionData": false
    }
  ]
}
```

**`GET /api/brands`** — the brand roster. Drives the top-bar brand switcher and the modal’s brand-filtering logic.

```json
{
  "data": [
    { "code": "PLS", "name": "PROLOOK" },
    { "code": "RDL", "name": "Riddell" },
    { "code": "ALLI", "name": "Alli" }
  ]
}
```

### 9.2 Production totals (drives the KPI cards and the main table)

**`GET /api/production-summary`**

Query params (all optional, sensible defaults applied server-side): `factoryIds[]`, `years[]`, `brandCode` (omit or `"ALL"` for no brand filter).

Returns monthly Approved and Shipped unit totals per factory per year — already filtered/scaled server-side to the requested brand, so the frontend never has to re-derive a brand’s proportional share of a factory’s units itself (that weighting logic currently lives in the mockup’s JS and is a backend-appropriate calculation, not something that belongs in the UI layer).

```json
{
  "data": [
    {
      "factoryId": "blb",
      "year": 2026,
      "approvedByMonth": [58451, 46355, 40186, 12719, 0, 0, 0, 0, 0, 0, 0, 0],
      "shippedByMonth":  [43816, 51711, 53102, 11873, 0, 0, 0, 0, 0, 0, 0, 0]
    },
    {
      "factoryId": "hcm",
      "year": 2026,
      "approvedByMonth": [7472, 11490, 12525, 3224, 0, 0, 0, 0, 0, 0, 0, 0],
      "shippedByMonth":  [8348, 11545, 11788, 3964, 0, 0, 0, 0, 0, 0, 0, 0]
    }
  ]
}
```

A month value of 0 for a not-yet-elapsed month (e.g. May 2026 onward, if today is still April 2026) should be distinguishable from a genuine zero for a month that’s already happened — the frontend renders both as “—” but this distinction may matter for the backend’s own data-quality checks. Consider an explicit `isFuture` flag or `null` (vs. `0`) for not-yet-elapsed months so the frontend doesn’t have to infer “the future” from today’s date plus a data value of zero, the way the current mockup does.

**`GET /api/kpi/year-over-year`**

Query params: `factoryIds[]`, `metric` (`"units"` or `"idv"`), `brandCode`.

Independent from `production-summary` because its comparison window is a fixed *basis* — current year’s elapsed months vs. the same months of the prior year (see section 3) — and does not follow the Year filter at all. Note this window is a moving target, not a hardcoded date range: the backend should compute “elapsed months” from where real data actually ends (e.g. the last month with a non-zero total), not from a fixed month baked into the mockup — that’s exactly the kind of ownership this endpoint is meant to take off the frontend. Keeping it a separate endpoint means the backend, not frontend arithmetic, owns that computation, which needs to advance every time a new month of data lands, not just once a year. The `metric` param is what lets this endpoint's numbers track the Units/IDV toggle while the comparison basis itself stays fixed.

```json
{
  "data": [
    {
      "factoryId": "blb",
      "currentPeriodLabel": "Jan–Aug 2026",
      "priorPeriodLabel": "Jan–Aug 2025",
      "currentValue": 330211,
      "priorValue": 375513,
      "percentChange": -12.1
    }
  ]
}
```

### 9.3 IDV item data (drives the item-breakdown modal)

**`GET /api/factories/{factoryId}/idv-catalog`** — the factory’s full item list with fixed IDV scores and each item’s typical order quantity. There’s no UI screen that displays this catalog on its own anymore (the mockup used to have a “catalog mode” for the modal, triggered from the table header — that trigger and mode were removed, see section 6.1) — but the data itself is still needed as the reference list the backend uses to proportionally split a period’s raw total across items for the one remaining modal mode (see `idv-breakdown` below).

```json
{
  "data": [
    {
      "itemName": "Compression Pant, Full Length, Men and Youth",
      "sports": ["APP"],
      "brands": ["PLS"],
      "idv": 1.5,
      "quantity": 1200
    },
    {
      "itemName": "Cinch Sack, Unisex",
      "sports": ["APP"],
      "brands": ["RDL", "ALLI"],
      "idv": 0.75,
      "quantity": 2100
    },
    {
      "itemName": "Champion Jersey, Men and Youth",
      "sports": ["SCR", "FB"],
      "brands": ["NWB"],
      "idv": 1.0,
      "quantity": 1300
    }
  ]
}
```

Note `sports` and `brands` are always arrays, even for a single value — this avoids the frontend needing a “could be a string or an array” check that the current mockup has to do (`asList()` in the mockup’s code exists specifically to paper over this; a real API shouldn’t require it).

**`GET /api/factories/{factoryId}/idv-breakdown`** — the “period mode” modal (triggered by clicking a data cell — see section 6.1).

Query params: `metric` (`"approved"` or `"shipped"`), `year`, `month` (omit month for a full-year breakdown, e.g. from clicking that factory’s cell in the bottom Totals row), `brandCode`.

```json
{
  "data": {
    "periodLabel": "March 2026",
    "totalUnits": 40186,
    "totalIdv": 88700.5,
    "items": [
      {
        "itemName": "Jersey, Reversible, 1-Ply, Full-Length, Men and Youth",
        "sports": ["FB"],
        "brands": ["NWB"],
        "idv": 2.25,
        "quantity": 8420,
        "idvTotal": 18945.0,
        "percentOfTotal": 21.4
      }
    ]
  }
}
```

The proportional-split math that turns a period’s raw total into individual item quantities (see section 6.1) is exactly the kind of business logic that should live server-side, not be recomputed in the browser from raw totals + a static catalog — it keeps the frontend a pure renderer of whatever the API returns, and keeps the rounding/remainder-distribution rules (so item quantities always sum back exactly to the period total) in one place.

### 9.4 Notes on scope

- None of the above endpoints exist yet — this section is a proposed contract only, meant to unblock frontend work while the real backend is designed, per section 11’s note on legacy data-source integration.
- If the external-stakeholder access idea in section 11 goes ahead, these same endpoints would need brand-scoped auth (e.g. a token whose allowed `brandCode` is enforced server-side, not just filtered client-side) — worth designing for from the start rather than retrofitting later.

## 10. Open question

Should the item-breakdown modal close when clicking outside it (on the dark overlay), in addition to the × button and Escape key?

Some component libraries deliberately disable backdrop-click-close on modals, to avoid accidental dismissal — so this isn’t necessarily a given just because it’s how the current design works, and it’s worth a deliberate decision either way rather than assuming.

## 11. Related items outside this document’s scope

These aren’t frontend/UI concerns, so they’re not reflected in the layout/behavior described above — noting them here so they don’t get lost:

- **Legacy data sources:** the data feeding this dashboard needs to integrate multiple legacy sources for historic values — the old difficulty-values page, the brand-agnostic difficulty-values page, and manual-orders data. This affects data completeness/accuracy but not the UI itself; the UI should keep handling “no data yet” the same way (blank/em-dash cells) regardless of which pipeline eventually fills a given cell.
- **External user permissions:** external brand stakeholders may eventually need secure access to a filtered version of this dashboard (e.g. locked to their own brand) without gaining access to the rest of the internal CorpSoft app. This is a hosting/access-control decision, not a layout change — but if it’s approved, the brand switcher (section 2.2) may need a “locked to one brand, can’t switch” mode for that audience. Worth keeping in mind so the brand-filter logic isn’t built in a way that assumes every viewer can always see “All Brands.”

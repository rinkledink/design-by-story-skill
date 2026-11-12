# Prototype visual spec

The shell (`assets/shell.html`) already implements the frames, switchers and story
panel. This file covers how the product inside the frames should look and behave.

## Non-goals

- No backend, API calls, routing library, build tooling or component framework.
- No real auth, no loading skeletons, no error states beyond friendly empty states.
- Target desktop viewing at ~1440px; the page itself needn't be responsive.

## Frames

| Device  | Frame | Size |
|---------|-------|------|
| Website | Browser chrome: traffic lights, back/forward/home, tab whose title follows the current screen, address bar whose path follows the current screen (`setWebUrl`). | Fills stage, max ~1280px |
| iPhone  | Dark bezel, ~54px radius, dynamic island, status bar (9:41, signal, 5G, battery), home indicator. | 392 × min(844px, 88vh) |
| Android | Dark bezel, ~40px radius, punch-hole camera, system-font status bar, gesture bar. App bar title in the sans font, not the serif. | 392 × min(844px, 88vh) |
| iPad    | Dark bezel ~14px, ~40px radius, status bar, home indicator. Portrait. | 760 × min(1200px, 92vh) |

Add or remove devices by editing `DEVICES` and the matching frame markup. All
frames are always in the DOM; switching only toggles visibility (~150ms fade), so
no state is lost.

## Design tokens

CSS custom properties on `:root`. Pick one distinctive brand colour per product;
keep neutrals warm (off-white bg reads more considered than pure white).

```css
--brand  --brand-deep  --brand-soft
--bg  --surface  --surface-2
--ink  --ink-soft  --ink-mute  --line  --line-strong
--ok  --ok-soft  --warn  --warn-soft  --err  --err-soft
--shadow-sm  --shadow-md  --shadow-lg
```

## Typography

Three Google Fonts, each with one job:

- **Fraunces** (400–600) — page titles, hero numbers, big stats.
- **Inter Tight** (400–700) — all UI: buttons, labels, body, table cells.
- **JetBrains Mono** — IDs, codes, dates in tables, numeric read-outs, and small
  ALL-CAPS labels (`9–10px`, `letter-spacing: .1em`).

## Website layout

- Left sidebar (~240px) with grouped sections, icon + label, active highlight.
- Top bar (~52px): breadcrumb left, primary actions right.
- Main: tables (sortable, status pills, avatar stacks, click-to-open rows),
  detail pages (header, summary card, tabs), multi-section create/edit forms.

## App layout (phones / tablet)

- App bar (~56px): big title + action buttons.
- Content is mostly cards: white surface, ~14px radius, subtle border; title
  15px semibold, metadata rows with icon + text, status pill on the right.
- Bottom tab bar when there are 3–5 top-level sections.
- Tapping a card opens a detail screen with a back button.

## Shared components

- **Status pills** — one style per entity state, reused everywhere
  (e.g. draft = grey, ready = brand-soft, in progress = brand, done = ok-soft).
- **Avatar stacks** — 26px initials circles, `margin-left:-6px`, white ring,
  `+N` chip after 3–4, full name in `title`.
- **Toasts** — bottom-centre, after any create/edit/assign, auto-dismiss ~2.4s.
  Toast inside the active frame, not the page.
- **Drawers** — right-side, ~420px, slide in over a dim backdrop (website only).
- **Bottom sheets** — the app equivalent of drawers.
- **Popovers** — anchored pickers; close on outside `mousedown` (not `click`,
  which misfires when the DOM re-renders mid-click).

## Icons

Inline SVG only, 24×24 viewBox, `fill="none" stroke="currentColor"
stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"`. Keep them in
one `I` object (`I.search`, `I.plus`, `I.chev`) and interpolate into templates.

## Rendering

- One shared `state` object; every device reads and writes it.
- Per-device UI state (current screen, open sheet, selected id) lives in
  `ui[device]`.
- `render(device)` redraws a whole frame from `state` + `ui[device]`.
  After any mutation call `renderAll()` so other devices are current when
  you switch. Manual, but fine for a prototype.

## Copy

Real, human, direct. Short labels. Empty states get a friendly one-liner. Help
text is concrete ("The days you'll be on site", not "Date range for this record").
All sample data is invented.

## Accessibility basics

Semantic buttons/links, visible focus, `aria-label` on icon-only buttons,
`aria-pressed` on toggles.

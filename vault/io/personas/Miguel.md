# Miguel — Touch Accessibility (cross-product)

Product: IO (origin), applies cross-product — see [[autolayout-ux]]
ORCA Role: workshop pending — cross-cutting accessibility constraint, not a
product actor

## Context

Miguel is an adult with a rare hand condition: his fingers are fused
(syndactyly-like), which limits fine independent finger movement and
precise small-target tapping. He uses touchscreens daily but needs **larger,
clearly separated** tap targets — he cannot reliably hit two controls that
sit edge-to-edge, and a mis-tap on a neighboring control is a real cost
(wrong filter, wrong nav item), not a minor annoyance.

He is not a persona for one product's feature — he is the concrete "why"
behind the autolayout no-glued-controls rule (`partials/autolayout-ux.md`),
same role Owner-Operator plays for readability.

## Goals / jobs

- Tap a filter, button, or nav item on the first try, without zooming in or
  triple-checking finger placement.
- Trust that adjacent controls (filter tabs, card grid items, toolbar
  buttons) have a real visible and touchable gap — not just a border.

## Frustrations

- Filter bars or tab rows rendered edge-to-edge ("glued") with no gap
  between touch targets.
- Controls that look separated (a border) but sit at zero actual spacing —
  the visual read and the touch reality disagree.
- Tap targets below a comfortable minimum size, especially in dense grids
  (card grids, chip rows).

## Scenario seed

> I open `/kit` to filter by agent. The filter tabs (ALL / AGENTS / SKILLS /
> SPELLS / PARTIALS) sit flush against each other. I aim for AGENTS and land
> on SKILLS instead — I have to retry, and on a bad day that costs real
> effort, not just a second of my time.

## Evaluation checklist (touch accessibility)

When reviewing the site as this persona:

1. **No glued controls** — any row of adjacent interactive elements (filter
   tabs, chips, nav items, toolbar buttons) has a real, computed gap ≥ 8px
   between hit areas — check computed layout, not just CSS source (a
   conflicting `display` or collapsed flex container can silently zero out
   an intended `gap`).
2. **Tap target size** — interactive controls ≥ ~44px in at least one
   dimension when feasible (`partials/portfolio-typography.md`).
3. **Borders are not enough** — a shared border between tabs (`border-right`
   only) does not count as separation; each control needs its own spaced
   hit area (see autolayout-ux anti-pattern note).
4. **Dense grids** — card grids and chip rows keep 8pt gap even when
   content is small (`.kit-card-grid`, `.kit-card__rels`).
5. **Hover/focus is not the only cue** — separation must be visible and
   present at rest, not only revealed on hover (Miguel taps, he doesn't hover).

## Appears in

- `partials/autolayout-ux.md` — motivating persona for the "never glue
  controls" rule, alongside Owner-Operator (readability)
- `usability-tester` agent — primary persona this agent tests against by
  default when no other persona is named
- Portfolio `/kit` filter bar — concrete bug found and fixed for this
  persona (missing gap from a conflicting `display` rule)

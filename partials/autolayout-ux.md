<!--
PARTIAL: Autolayout + atomic UX contract (Figma mental model → CSS/React).
Link from: ux-engineer, ui-designer, interaction-designer, persona-crafter.
Tool-agnostic — applies to Portfolio, Deviante, Figma ADS instances.
-->

# Autolayout UX (atomic + 8pt)

**Principle:** every screen is **containers inside containers**, not loose
elements on a canvas. Think Figma **Auto Layout**: direction, gap, padding,
alignment, wrap, fill/hug — implemented with flex/grid + tokens.

Personas from `persona-crafter` **cannot read or touch** screens with glued
controls, clipped tags, or huge dead zones — Owner-Operator (readability)
and **Miguel** (`vault/io/personas/Miguel.md` — rare hand condition, needs
real spaced tap targets, not just a border between controls) are the
concrete "why". `ux-engineer` audits for them; `ui-designer` fixes static
chrome; `interaction-designer` fixes motion; `usability-tester` runs the
structured persona check before ship.

## 8-point spacing

All gaps and padding use **multiples of 8px** (4px only for hairlines/borders).

| Token | px | Use |
|-------|-----|-----|
| `--cases-space-8` | 8 | Tight cluster (filter chips, tag rows) |
| `--cases-space-16` | 16 | Section stack, card padding |
| `--cases-space-24` | 24 | Major blocks (intro → toolbar → body) |
| `--cases-space-32` | 32 | Page sections |

No arbitrary `0.65rem`, `0.85rem`, `gap: 0` between sibling controls unless
documented exception in a component spec.

## Atomic hierarchy

```
Atom     — button, beacon, tag, input, label
Molecule — filter chip, card head, search field
Organism — filter bar, card grid, detail pane, split layout
Template — `/kit` list + `/kit/[kind]/[slug]` detail; `/cases` shell (panel)
```

**Rule:** organisms are **named containers** with explicit autolayout — never
“style the `<ul>` and hope.”

## Figma Auto Layout → CSS

| Figma | CSS pattern | Class prefix (Portfolio) |
|-------|-------------|---------------------------|
| Vertical stack, gap 16 | `flex-direction: column; gap: var(--cases-space-16)` | `.kit-stack`, `.kit-stack--gap-16` |
| Horizontal cluster, wrap, gap 8 | `display: flex; flex-wrap: wrap; gap: var(--cases-space-8)` | `.kit-cluster`, `.kit-cluster--gap-8` |
| Split pane (fill) | `display: grid; grid-template-columns: 1fr 1fr; gap: 16` | reserved — kit uses **route** list↔detail, not split |
| Grid auto-fill | `grid-template-columns: repeat(auto-fill, minmax(...))` | `.kit-card-grid` |
| Hug contents | `width: fit-content; max-width: 100%` | filter cluster |
| Fill container | `min-width: 0; flex: 1` | detail pane, prose columns |

**Anti-pattern:** one shared border around **many** tabs with `border-right`
only — reads as “glued.” Prefer **separate bordered chips** in a cluster with
`gap: 8`.

## Proximity & readability

1. **Related controls cluster** — filters + counts together; search on same
   toolbar row or next stack step (16px), not orphaned.
2. **Split layouts** — list + detail both **framed** (border/padding). Empty
   detail is a **visible panel**, not bare text floating in void.
3. **Card grid** — uniform gap 8; titles truncate with **min padding** 16
   inside card; tags never clip without `title` tooltip.
4. **Responsive** — stack split → single column `< 960px`; cluster wraps;
   grid `auto-fill` reflows. Content order: intro → toolbar → list → detail
   (mobile: detail below list or route-only).

## Review workflow (ux-engineer)

1. Load persona checklist if owner named one (`vault/.../personas/`).
2. Open live URL + screenshot.
3. Audit: spacing (8pt) | clusters | dead space | truncation | contrast.
4. File issues with selector + suggested autolayout fix.
5. Hand static fixes → `ui-designer`; motion → `interaction-designer`.

## Portfolio hot spots

- `/kit` — filter bar, card grid, detail empty state (reference implementation
  of `.kit-stack` / `.kit-cluster` / `.kit-split`). **Fixed 21/07/2026:**
  `.kit-filter` carried a second `display: contents` rule that silently
  overrode the `.kit-cluster` flex + `gap: 8px` on the same element (equal
  specificity, later rule wins) — filters rendered edge-to-edge with zero
  real gap despite the gap token being "correct" in source. When a gap looks
  right in CSS but glued in the browser, check for a **second class on the
  same element** overriding `display`.
- `/cases` — UC row grid, folio toolbar (also 8pt via `--cases-space-*`).

## Boundaries

- Does not replace ADS/Figma component SoT — **instances** follow this layout.
- Does not invent new spacing tokens outside the 8pt scale without architect yes.

<!--
PARTIAL: Portfolio (IO) typography contract — link from ui-designer, do not paste.

Referenced from:
- gestalt-kit/agents/ui-designer.md
- portfolio/app/portfolio-tokens.css (tokens implement this)
-->

# Portfolio typography — Carbonot + one chrome size

**Product:** Portfolio / alander.io only (IO). Deviante themes separately.

## Carbonot — exactly two sizes

| Role | Token | Used for |
|------|--------|----------|
| **Display** (large titles only) | `--carbonot-display-size` (= `--type-display-size`) | Brand name in header, hero titles like **currently @TCEPR**, page `h1` |
| **Chrome** (everything else Carbonot) | `--carbonot-size` | Tables, UC codes, nav chrome, labels, filters, buttons, stamps |

Do **not** invent a third Carbonot size (`calc(… * 1.12)`, one-off clamps, separate “cap” sizes). Cap/label aliases must equal `--carbonot-size`.

## Readable copy — GT Planar (prose)

Long-form / table cell body / UC titles that must be read as sentences → `--font-prose` + `--prose-body-*` / `--cases-read-*` (same body size family). Not a third Carbonot scale.

**Mobile / WebKit:** GT Planar VF must stay upright. Register an italic `@font-face` that points at the **same** upright file; set `font-style: normal`, `font-synthesis: none`, and `font-variation-settings: 'slnt' 0` on reading surfaces. Do not rely on browser italic synthesis. Cases read size = `--prose-body-size`, never `--carbonot-size`.

## Accessibility & consistency (ui-designer)

- Prefer **color / weight / border** over **opacity** for primary actions (opacity alone fails contrast).
- Focus visible on interactive controls; tap targets ≥ ~44px when feasible.
- Color: stick to Arcadia + work accents in `portfolio-tokens.css` — no orphan hex when a variable exists.
- Minimize font-size variants sitewide; if a new size seems needed, ask whether display vs chrome vs prose already covers it.

## SoT

- **Rule text:** this partial.
- **Implementation:** `portfolio/app/portfolio-tokens.css` + `globals.css`.
- Drift (extra Carbonot sizes, float controls over copy) → fix tokens/CSS; update this partial only if the owner changes the contract.

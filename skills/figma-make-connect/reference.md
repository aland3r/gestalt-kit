# Figma Make connect

## Why this exists

Before 21/07/2026, the only way to bring a Figma Make screen into a repo was:
export → download a zip from the browser → drop the folder into the repo →
read through it by hand. That's what produced `Process Mining Canvas Design
by Figma/` in this repo — and it has to be repeated by hand every time the
owner iterates on the Make file. This skill replaces the manual step: the
**Figma MCP already available in this environment can read a Figma Make
file's live source directly**, no download required.

## The one fact that makes this work

`get_design_context` (Figma MCP) explicitly supports Figma Make URLs:

> If the URL is of the format `https://figma.com/make/:makeFileKey/:makeFileName`
> then use the `makeFileKey` to identify the Figma Make file. Only for Figma
> Make files (URLs containing `/make/`), and only when calling
> `get_design_context`, assume the `nodeId` is `0:1`.

So for a URL like `https://www.figma.com/make/ZZKdwxgmeCNJFG64zGbADe/Process-Mining-Canvas-Design`:
- `fileKey` = `ZZKdwxgmeCNJFG64zGbADe` (the segment right after `/make/`)
- `nodeId` = `0:1` (always, for Make files — not read from the URL)

The call returns resource links to **every source file in the Make project
live** (`App.tsx`, every `components/ui/*.tsx`, styles, `package.json`,
referenced images) — the current state, whatever the owner last edited in
Figma Make, no zip in between.

**`get_metadata` and `get_screenshot` do NOT work on Make files** — only
`get_design_context` does. Don't reach for them here.

## Procedure

1. **Load `figma-design-to-code` first — mandatory, not optional.** Read
   `skill://figma/figma-design-to-code/SKILL.md` via the Figma MCP's
   `get_figma_skill` tool before calling `get_design_context`. It defines the
   gate protocol (G1 design-context → G2-G4 pre-edit → G5 asset fidelity) and
   the priority order for hints (Code Connect > doc links > design
   annotations > tokens > raw hex). Pass
   `skillNames="figma-design-to-code"` (or `resource:figma-design-to-code`
   if loaded as an MCP resource) on the `get_design_context` call itself —
   it's a logging parameter, but it's part of following the skill correctly.
2. **Call `get_design_context`** with the extracted `fileKey` and
   `nodeId: "0:1"`. Treat the returned code as **reference, not final** — it's
   React + Tailwind (TSX), and this repo's frontends are plain JS/JSX.
3. **Reuse before porting.** Check `<product>/web/src/components/ui/` for a
   shadcn primitive that's already been ported (button, card, dialog, etc. —
   Deviante already has 12+ from the 21/07 Tailwind adoption) before copying
   the Make file's version again. Same for tokens: check
   `<product>/web/src/index.css` before re-declaring colors already defined
   there.
4. **De-type on the way in.** Strip `interface`/`type` declarations, generic
   `useState<T>`/`useRef<T>` params, and prop-type annotations — mechanical,
   not a redesign. See `deviante/web/src/pages/ProcessMiningPreviewPage.jsx`
   and `DashboardPage.jsx` for the pattern already established.
5. **Follow the G5 asset rule** — icons/images come back as exported assets
   with expiring URLs (~7 days); download the actual bytes for anything that
   ships in a commit, don't leave a live Figma asset URL in product code.

## Boundary: Figma Make ≠ the canonical design-system Figma file

This skill is **not** a replacement for the governed ADS → product-library
chain described in `deviante-figma-use-cases` (or the equivalent for other
products). A Figma Make file is an informal, AI-generated prototype — useful
as a fast reference or starting point, but it doesn't pass through the
ADS-library discipline that the canonical Figma file does.

If a Make file's direction is meant to **become** the product's real visual
identity (as happened with `ZZKdwxgmeCNJFG64zGbADe` for Deviante on 21/07 —
see `sprint-plan-2026-07.md` § "Governança Figma"), that's an explicit owner
call, not something this skill decides on its own — flag it, don't assume it.

## Known Figma Make files in active use

| File | Product | fileKey |
|------|---------|---------|
| Process Mining Canvas Design | Deviante | `ZZKdwxgmeCNJFG64zGbADe` |

Add a row here when a new Make file becomes a real reference for a product —
this table is the fast lookup so nobody has to re-paste the URL from chat.

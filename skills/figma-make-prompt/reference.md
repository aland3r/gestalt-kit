# Figma Make prompt — request instead of freehand

**Governance this skill serves:** [sot-matrix.md § Figma is always the
design reference](../../partials/sot-matrix.md) — design decisions belong
to the owner + design team, an agent proposes, it does not decide. This
skill is the practical move **when the Figma reference doesn't exist yet**:
draft a request, don't invent pixels.

## Why (token economy)

Freehanding a new screen's visual language (guessing colors, spacing,
component shapes) costs a full design pass in-chat, then usually a revise
cycle once the owner/design team compares it against the product's real
Figma file — two or three expensive turns for something a text prompt to
Figma Make can produce in one. Drafting a grounded prompt instead is one
focused turn: check the gap, quote real tokens, hand off, implement from
what comes back.

## When to use

- A UC or screen is confirmed on the esteira (`uc-esteira.md`) and ready to
  build, but **no Figma frame exists** for it yet in the canonical product
  file's `use cases` page.
- The owner asks for a new screen and a Figma MCP is connected, but the
  screen isn't drawn anywhere reachable.
- Not for screens that already have a frame — call `get_design_context`
  directly on that frame instead (see `deviante-figma-use-cases`).

## Procedure

1. **Confirm the gap, don't assume it** — `get_metadata` on the canonical
   file's `use cases` page (no nodeId → lists top-level pages/frames).
   Search by UC name/number before concluding nothing exists.
2. **Ground the chrome in something real** — pull an already-drawn frame in
   the *same* file via `get_design_context` (or `get_metadata` first if the
   node id isn't known) and quote its exact values: hex colors, font
   weights, spacing, component names. Never invent an adjacent color or
   "close enough" token — if the real frame says `#1b1b1b` / `#161365`,
   the prompt says that, not a guess at "similar dark blue."
3. **Ground the content in the confirmed spec** — load the UC row + steps
   from live `portfolio.use_cases` / `use_case_steps` (Supabase MCP). Screen
   sections, fields, and CTAs in the prompt must trace to actual
   `description_what` / step `actor_action` / `system_response` text, not
   invented flows.
4. **Write the prompt in the established shape** — same
   Contexto / Objetivo / Regras / Entregável structure already used in
   [design-system/README.md § Prompts prontos](../../../design-system/README.md)
   so it reads the same way for a human or for Figma Make. Include:
   - Chrome block: exact values copied from step 2, explicit "not X" lines
     for anything the product has used elsewhere that doesn't apply here
     (e.g. a different product's accent color).
   - Screens block: one subsection per screen, grounded in step 3.
   - Rules: which existing component set to reuse (don't invent a new kit),
     typography/dark-only/no-invented-states constraints.
   - Entregável: what bundle shape you need back (so it's consumable the
     same way as prior Figma Make exports — React + Tailwind +
     `components/ui/*`).
5. **Hand it to the owner** — they paste it into Figma Make themselves.
   Do not call a Figma write/generate tool on the owner's behalf unless they
   explicitly ask you to submit it directly.
6. **Implement from what comes back** — once the owner shares the
   resulting Make file link/fileKey, pull it with `get_design_context`
   (Figma Make files: `nodeId` is always `0:1`; `get_metadata`,
   `get_screenshot`, and `get_variable_defs` are **not** supported for
   `/make/` URLs — only `get_design_context` works). Compare against the
   Rules block before writing code; flag drift instead of silently
   reconciling it yourself.

## Boundaries

- Does not replace `deviante-figma-use-cases` — once a Make-generated
  concept is approved, moving it into the canonical file's `use cases` page
  as real frames still means only screen frames + instances of the
  Deviante library (that skill's structural rule), not raw Make output
  pasted in.
- Does not authorize writing to any Figma file — request/read only, unless
  the owner explicitly asks for a write.
- Does not skip the esteira gate — draft the prompt only for a UC that is
  already `spec_confirmed`, not to speculate on scope.

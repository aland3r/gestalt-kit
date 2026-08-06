<!--
SCAFFOLD, not a finished agent. Fields are set to sensible defaults;
the sections under "Fill this in" are yours — that's the point. This
file teaches the shape; you write the judgment calls.
-->
---
name: deviante-frontend
description: >-
  EDIT ME. One sentence on what this agent builds (deviante/web, React/Vite)
  and one sentence on when Claude should reach for it vs. just chatting.
  Example shape: "Builds and edits deviante/web React/Vite screens against
  documented use cases. Use when implementing a Deviante UC's UI, wiring a
  page to the API client, or fixing a component bug in deviante/web."
model: sonnet
skills: deviante-domain, dev-quest-hud, gestalt-context
# disallowedTools: Bash   # uncomment once you decide this agent shouldn't run shell commands
---

## Scope (fill this in)

- Owns: `deviante/web/src/` — pages, components, `lib/api.js` client calls.
- Does not own: `deviante/api/` (that's `deviante-backend`), schema files
  (that's `database-integrations`).
- *Example line to adapt:* "Every new page reads its use case in
  `gestalt-kit/vault/products/deviante/user stories/` first — list the
  `DV-UC{N}-AC{M}` acceptance criteria before writing a component."

## Conventions to enforce (fill this in)

- *Prompts to answer yourself:* Which UI kit / CSS approach does
  `deviante/web` already use? Any component you always reuse (loading
  states, error banners)? Any pattern from `dev-quest-hud` you want every
  page to respect (e.g. never touch Quest Log rendering in prod builds)?

## Boundaries — what this agent should NOT do (fill this in)

- *Starting point:* should it ever edit `deviante/api/`? Touch
  `data/schema/`? Call Supabase directly instead of going through
  `lib/api.js`? Write these down explicitly — an agent without boundaries
  will happily "helpfully" cross them.

## How to test it once it's real

Ask it to implement one small, already-documented UC step and see if the
result matches what you'd have written by hand. Adjust the description
first if it doesn't get invoked when you expect (see
`gestalt-kit/README.md` → "How to add a new skill" — the same
troubleshooting applies to agent descriptions).

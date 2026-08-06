---
name: deviante-backend
description: >-
  Implements Deviante API routes and persistence in deviante/api (Kotlin/Ktor,
  Exposed, OOP). Use when adding a Ktor route, an Exposed table/repository, or
  wiring a use case's acceptance criteria to a real endpoint — not a mock, and
  not persistence logic written in JS/TS.
model: sonnet
skills: deviante-domain, gestalt-database, implement-deviante-uc
# disallowedTools:   # decide later — this agent likely does need Bash for gradle
---

## Scope

- Owns: `deviante/api/src/main/kotlin/` — routing, repositories, tables.
- Existing pattern to follow: `repository/UserRepository.kt` +
  `db/UsersTable.kt` — new repositories (managers, processes, event logs,
  drift results, ...) should look like siblings of these, not reinvent the
  shape.
- Does not own: `deviante/web/` (that's `deviante-frontend`), schema *design*
  decisions that affect other products (that's `database-integrations` —
  this agent implements a schema once it's agreed, doesn't invent one solo
  for something as consequential as event-log/drift tables).

## Hard rule — Kotlin owns persistence, not JS

**Deviante is OOP-first via Kotlin end to end for business/data logic.**
`deviante/web` (TS/React) calls the Ktor API; it does not talk to Postgres
directly and does not reimplement persistence or business rules in
TypeScript. **Portfolio is the sole exception** in this monorepo (it may use
`supabase-js` directly) — do not generalize that pattern into Deviante.

**This is a standing assurance, not a one-time check.** On **every** task
that touches Deviante data — a new UC, an edit to an existing one, a bug
fix — actively confirm the actual manipulation happens through Kotlin
classes (`*Table` + `*Repository`, Exposed), not just that a route exists.
Re-verify each time; do not assume a prior pass covered it.

Before wiring any route or feature to real data:

1. **Check** whether a Kotlin persistence layer already exists for that data
   (`*Table` + `*Repository` under `deviante/api/src/main/kotlin/`).
2. **If missing, create it** in Kotlin, following the `UsersTable` /
   `UserRepository` shape — do not stub the data in the frontend, mock it in
   JS, or leave the route half-wired to fake persistence "for now."
3. Only after the Kotlin layer exists does the route/service call it.

See [architecture.md § Endorsed patterns](../docs/architecture.md) (row
"Deviante stack boundary") — the cross-agent version of this rule.

## Conventions to enforce (fill this in)

- *Prompts to answer yourself:* Exposed DSL conventions you want kept
  consistent (naming, nullable columns, `REFERENCES` style already used in
  `data/schema/deviante/`)? Error-handling pattern across routes? Where do
  `DV-UC{N}-AC{M}` acceptance criteria map to specific endpoints — should
  the agent list them as a checklist before marking a route "done"?

## Boundaries — what this agent should NOT do (fill this in)

- Never write persistence or business-rule logic in `deviante/web` (JS/TS) —
  see Hard rule above.
- *Still open:* should it run `gradle` tasks itself, or just write code
  and let you run/verify? Should it touch `data/schema/*.sql` files
  directly, or only Kotlin code that assumes a schema already applied
  manually in DataGrip (current convention — see `gestalt-database` skill)?

## How to test it once it's real

Same as `deviante-frontend`: give it one small, already-scoped endpoint and
compare the result to what you'd write by hand before trusting it with a
bigger slice (e.g. the drift job).

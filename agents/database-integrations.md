<!--
SCAFFOLD, not a finished agent. This is the one worth being most careful
with — schema and cross-product decisions are the hardest to undo. Treat
its early output as a proposal to review, not an approved migration.
-->
---
name: database-integrations
description: >-
  EDIT ME. One sentence on scope (schema design across data/schema/, Supabase
  Auth/Postgres integration, cross-product data flow) and one sentence on
  when to use it vs. deviante-backend. Example shape: "Designs and reviews
  PostgreSQL schema in data/schema/{product}/ and Supabase Auth/integration
  wiring. Use when adding a new table, changing a schema shared across
  products, or deciding how event logs / traces / drift results should be
  modeled — not for implementing a single already-agreed endpoint."
model: sonnet
skills: gestalt-database, seed-accounts, deviante-domain
# disallowedTools:   # strongly consider disallowing Bash and anything that
#                      could apply a migration automatically — this agent
#                      should propose SQL, not run it, per current
#                      "apply manually in DataGrip" convention.
---

## Scope (fill this in)

- Owns: schema *proposals* in `data/schema/{product}/`, cross-schema FK
  conventions (`deviante.*`, `milebrick.*`, `portfolio.*` on one database),
  and reasoning about how Supabase Auth ties to each product's user table.
- This is the agent for the hardest open question right now: how do
  `event_logs`, `operations`, `traces`, and drift results relate to each
  other and to `deviante.processes`? That's a modeling decision for your
  thesis, not something to accept without reading — this agent should
  explain its reasoning, not just hand you DDL.
- Does not own: writing the Kotlin that uses the schema once it's agreed
  (that's `deviante-backend`).

## Conventions to enforce (fill this in)

- *Prompts to answer yourself:* Exposed/Kotlin type mapping conventions?
  Required indexes for the volumes you expect (event logs can be large)?
  Naming: singular vs. plural tables (`deviante.equipment` is already
  singular by convention — is that intentional going forward?).

## Boundaries — what this agent should NOT do (fill this in)

- *Starting point:* never apply a migration itself (write the `.sql` file,
  let you review and run it in DataGrip — matches the existing "no Flyway"
  convention). Never invent a schema for a new product without checking
  `gestalt-kit/skills/gestalt-database/reference.md` first.

## How to test it once it's real

Give it the exact modeling question from the sprint ("how should traces,
operations, and drift results relate?") and see if its proposal matches
your own mental model before it touches any file.

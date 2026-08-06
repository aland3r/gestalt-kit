# Implement Deviante Use Case

Turn a **owner-confirmed** use case into working code in `deviante/api/` and
`deviante/web/`.

**Gate (mandatory):** follow
[partials/uc-esteira.md](../../partials/uc-esteira.md). Do **not** start API/web
work until the owner has explicitly confirmed the **spec card** loaded from
live `portfolio.use_cases`. Coding without that yes is forbidden.

## Prerequisites

1. **Esteira gate passed** — `metadata.esteira.review_status` is
   `spec_confirmed` (or owner just said yes in-session and you wrote the mark
   via MCP). Spec card ACs are the checklist.
2. UC loaded from **DB via Supabase MCP** (preferred). If MCP missing: stop;
   ask owner to link MCP or paste from `/cases` — do not invent from memory
   or assume vault is current.
3. Dependency use cases are implemented or stubbed.
4. Read [deviante-domain.md](../deviante-domain/reference.md) for entity
   relationships.
5. Personas for the actor: linked or workshopped (`persona-crafter`) before
   UI copy invents a cast.

Vault files under `gestalt-kit/vault/products/deviante/user stories/` are
**replicas** — useful for reading prose, not SoT. On vault↔DB drift, **DB
wins** unless the owner says otherwise.

## Workflow

```
Task Progress:
- [ ] Esteira: load UC from DB + show spec card
- [ ] Owner explicit yes (mark metadata.esteira if MCP available)
- [ ] List confirmed acceptance criteria as implementation checklist
- [ ] Identify API endpoints / data models needed
- [ ] Identify web screens / components needed
- [ ] Implement API first (Ktor)
- [ ] Implement web UI (React)
- [ ] Verify each acceptance criterion against confirmed card
- [ ] Gamifier: sync quests/steps to valid UC scope (see gamifier skill)
- [ ] Optionally /ship-quest for fine-grained quests; set esteira accepted
```

## Step 0 — Esteira hard stop

If the owner has not confirmed:

```text
BLOCKED: UC not owner-confirmed.
Next: /uc-gate {abp_id}  or  load spec card from DB and ask for yes.
```

Do not “start a little bit” of implementation while waiting.

## Step 1 — Read the confirmed UC

From live DB (and requirements / steps tables), extract:
- Pre/Post-conditions → API guards and state guarantees
- Steps → UI flows and endpoint triggers
- Acceptance criteria → test cases

## Step 2 — Split API vs Web

| Concern | API (`deviante/api/`) | Web (`deviante/web/`) |
|---------|------------------------|------------------------|
| Validation (file size) | Server-side enforcement | Client-side feedback |
| Data persistence | PostgreSQL via `deviante.*` tables | API calls only |
| File parsing (PM4Py) | Background service | Upload UI + progress |
| ML / drift analysis | Computation service | Results display |
| Auth / sessions | Token/session management | Login forms, redirects |

**Rule:** Every acceptance criterion marked with validation or data integrity must be enforced server-side.

**No clutter:** Do not ship disabled buttons, placeholder providers, or "em breve" controls. If a feature is out of scope for the target version, omit it from the UI entirely. See [gestalt-context.md](../gestalt-context/reference.md).

### Database changes

1. Edit SQL in `data/schema/deviante/` (see [database.md](../gestalt-database/reference.md))
2. Apply in **DataGrip** — no Flyway/migrations in `deviante/api/`
3. Qualify every table: `deviante.users`, `deviante.managers`, etc.
4. Foreign keys: `REFERENCES deviante.users (id)`

## Step 3 — API Implementation

Follow existing Ktor structure:

```
deviante/api/src/main/kotlin/
├── main.kt
├── Routing.kt        # Add routes here
├── Http.kt
└── Serialization.kt
```

For each use case:
1. Define request/response models
2. Add routes in `Routing.kt`
3. Implement handler logic
4. Add test in `src/test/kotlin/`

Run: `./gradlew test` and `./gradlew run`

## Step 4 — Web Implementation

Follow existing React/Vite structure:

```
deviante/web/src/
├── App.jsx             # Routing (add routes as app grows)
├── components/         # Feature components
└── pages/              # Page-level views
```

For each use case:
1. Create page/component matching the steps table
2. Wire API calls
3. Implement UI states (loading, error, success) per black-box responses

Run: `npm run dev`

## Step 5 — Verify Acceptance Criteria

Map each `DV-UC{N}-AC{M}` to a verifiable check:

| AC type | Verification |
|---------|-------------|
| Validation | Unit test with invalid input |
| Uniqueness | Integration test with duplicate |
| UI feedback | Manual or component test |
| Data integrity | API test for cascade/delete |

## Step 6 — Gamifier after accept

When ACs pass and the owner accepts:

1. Set `metadata.esteira.review_status` → `accepted` (MCP).
2. Follow [gamifier](../gamifier/reference.md) — align implementation quests
   to the **confirmed** UC scope; do not invent quests outside that scope.
3. Optionally `/ship-quest` for fine-grained rows; UC `status`
   `ready`/`shipped` flips the auto `-spec` quest.

## Implementation Order

Follow use case dependency order (see [deviante-domain.md](../deviante-domain/reference.md)),
but **only** for UCs that have cleared the esteira gate:

1. UC1 → UC2 → UC3 → UC4 (foundation)
2. UC5 → UC6 (mapping)
3. UC12 → UC13 → UC7 (analysis)
4. UC8–UC11 (investigation extensions)
5. UC14 → UC15 (maintenance loop)

## Conventions

- Do not invent UI steps not on the confirmed spec card
- Do not skip acceptance criteria
- Keep changes minimal per use case
- Match code style of existing files in each repo
- Commit to the product repo (`deviante/api` or `deviante/web`), not gestalt root
- Same gate applies to any **IO** implement path: load from DB → card → yes → code

## Key Paths

| Resource | Path |
|----------|------|
| Esteira contract | `gestalt-kit/partials/uc-esteira.md` |
| UC SoT | Live `portfolio.use_cases` (+ steps, requirements) |
| Vault replica | `gestalt-kit/vault/products/deviante/user stories/` |
| API entry | `deviante/api/src/main/kotlin/Routing.kt` |
| Web entry | `deviante/web/src/App.jsx` |
| API config | `deviante/api/src/main/resources/application.yaml` |
| Database DDL | `data/schema/deviante/` |
| DB conventions | [database.md](../gestalt-database/reference.md) |

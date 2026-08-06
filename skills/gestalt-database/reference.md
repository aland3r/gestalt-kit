# Gestalt Database

Single PostgreSQL database shared by all Gestalt products. Each product owns a **schema**; tables never live in `public` for product data.

**Host (production + dev):** [Supabase](https://supabase.com) PostgreSQL — one project, database name `postgres`, **four schemas** (`deviante`, `milebrick`, `harpia`, `portfolio`). **Authentication:** Supabase Auth (e-mail/password, Google OAuth) — not custom bcrypt login in the API.

**Tool:** [DataGrip](https://www.jetbrains.com/datagrip/) — connect to the Supabase connection string; run DDL from `data/schema/` against the same database.

## Supabase layout

| Piece | Where |
|-------|--------|
| **Postgres** | Supabase project → Settings → Database → connection string (URI or JDBC) |
| **Auth users** | Supabase Auth (`auth.users`) — create/invite in dashboard or sign-in UI |
| **App data** | `deviante.*`, `milebrick.*`, `harpia.*`, `portfolio.*` schemas |
| **Web login** | `@supabase/supabase-js` + `VITE_SUPABASE_URL` + `VITE_SUPABASE_ANON_KEY` |
| **API (Ktor)** | JDBC to same Supabase Postgres (`DATABASE_JDBC_URL`, `DATABASE_PASSWORD`) |

`deviante.users.password_hash` stays **NULL** — credentials live in Supabase Auth. App tables use `id` = Supabase Auth user UUID where applicable.

### Where to put secrets (never commit)

| File | Variables |
|------|-----------|
| `deviante/web/.env` | `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` |
| `deviante/api/.env` or shell env | `DATABASE_JDBC_URL`, `DATABASE_USER`, `DATABASE_PASSWORD` |

Copy from `deviante/web/.env.example` and `deviante/api/.env.example`.

### JDBC example (Supabase)

**Session pooler** (recommended for Ktor / IDE / DataGrip):

```
jdbc:postgresql://aws-1-sa-east-1.pooler.supabase.com:5432/postgres?sslmode=require
```

User: `postgres.<project-ref>` (e.g. `postgres.ydjtrcjxhtmygytmebrk`).

**Direct** (IPv6 / no pooler):

```
jdbc:postgresql://db.<project-ref>.supabase.co:5432/postgres?sslmode=require
```

User: `postgres`.

Password = **database password** from Supabase → Settings → Database (not the publishable/anon key).

### Dev/test users

1. Create user in **Supabase Auth** (dashboard → Authentication → Users, or invite).
2. Seed **profile** rows in `deviante.managers` (and product tables) with the **same UUID** as `auth.users.id`.
3. Distribute login to testers via Supabase credentials — no self-service register UI in app v1 dev.

See [seed-accounts.md](../seed-accounts/reference.md).

## Layout

```
data/
├── datasets/              # LOCAL ONLY — event logs, CSV (see datasets/README.md)
│   └── deviante/
│       ├── manifest.json  # demo subset for UX (tracked if folder is versioned)
│       ├── manufacturing/ # .xes — gitignored
│       └── real/          # Prod1Torno.csv — gitignored
├── schema/
│   ├── deviante/          # DV tables (UC-driven)
│   │   ├── schema.sql
│   │   ├── users.sql
│   │   ├── managers.sql
│   │   └── ...
│   └── milebrick/         # MB tables
│       ├── schema.sql
│       ├── languages.sql
│       ├── users.sql
│       ├── vocabulary/
│       ├── context/
│       └── practice/
└── seed/
    ├── deviante/
    └── milebrick/seed.sql
```

**Source of truth:** SQL files in `data/schema/{product}/`, not Flyway or per-repo migration folders.

Apply scripts in **DataGrip** (or psql) against the shared database.

## Schema map

| Schema | Product | Code | Status |
|--------|---------|------|--------|
| `deviante` | Deviante | DV | Active (PIBITI) |
| `milebrick` | Milebrick | MB | Deferred |
| `harpia` | Harpia | HA | Deferred |
| `portfolio` | Portfolio | IO | Active (platform + site CMS) |
| `public` | Shared infra only | — | Avoid product tables here |

## Naming rule (mandatory)

**Always qualify table names with the product schema** in SQL, ORM mappings, and API queries:

```sql
-- correct
SELECT id, email FROM deviante.users WHERE email = $1;
INSERT INTO deviante.managers (user_id, full_name)
VALUES ($1, $2);

-- wrong
SELECT id, email FROM users WHERE email = $1;
ALTER TABLE managers ADD COLUMN user_id UUID REFERENCES users(id);
```

Foreign keys must also use the qualified name:

```sql
REFERENCES deviante.users (id)
```

## Deviante schema (current)

| Table | UC | State |
|-------|-----|-------|
| `deviante.users` | UC1 | Auth identity |
| `deviante.managers` | UC1 | Manager profile (1:1 with users) |
| `deviante.processes` | UC2 | Process metadata incl. `company_name` |
| `deviante.activities` | UC3 | Skeleton |
| `deviante.equipment` | UC14+ | Skeleton |

**Note:** table name is `equipment` (singular), not `equipments`.

### Attribute reference

Review of every `deviante.*` column — source of truth is `data/schema/deviante/*.sql`.

#### `deviante.users` (auth identity)

| Column | Type | Required | Notes |
|--------|------|----------|-------|
| `id` | UUID | ✓ | PK |
| `email` | VARCHAR(255) | ✓ | Unique; login identifier |
| `password_hash` | VARCHAR(255) | — | NULL when Supabase Auth / OAuth owns credentials |
| `created_at` | TIMESTAMPTZ | ✓ | |
| `updated_at` | TIMESTAMPTZ | ✓ | |

#### `deviante.managers` (Manager profile)

| Column | Type | Required | Notes |
|--------|------|----------|-------|
| `id` | UUID | ✓ | PK |
| `user_id` | UUID | ✓ | FK → `users`, unique (1:1) |
| `full_name` | VARCHAR(255) | ✓ | Display name |
| `created_at` | TIMESTAMPTZ | ✓ | |
| `updated_at` | TIMESTAMPTZ | ✓ | |

**Not on Manager:** company name (→ `processes`), country/city (deferred),
language/location profile fields (removed 21/07/2026 — copy-pasted from an
earlier Milebrick design, never valid for Deviante's actor).

#### `deviante.processes`

| Column | Type | Required | Notes |
|--------|------|----------|-------|
| `id` | UUID | ✓ | PK |
| `manager_id` | UUID | ✓ | FK → `managers` |
| `name` | VARCHAR(100) | ✓ | Process label; max 100 chars |
| `company_name` | VARCHAR(255) | ✓ | Owning organization for this workflow |
| `description` | TEXT | ✓ | Default empty string |
| `sector` | VARCHAR(100) | ✓ | Default empty string |
| `created_at` | TIMESTAMPTZ | ✓ | |
| `updated_at` | TIMESTAMPTZ | ✓ | |

#### `deviante.activities` (skeleton — UC3)

| Column | Type | Required | Notes |
|--------|------|----------|-------|
| `id` | UUID | ✓ | PK only today |

**Correction (owner, 21/07):** Activities are **shared across processes**,
not owned by one — do **not** extend with a simple `process_id` FK (that
was this doc's earlier assumption, now wrong). An Operation cannot exist
without attaching to an Activity first. This needs a many-to-many shape
(activity catalog + a process↔activity junction, since drift analysis
queries a *process's* activity usage, not the global definition) —
`architect` + `database-integrations` design this before UC3 schema ships,
per the sprint's "schema antes de código" principle. Do not invent the
junction table here without that pass.

#### `deviante.equipment` (skeleton — needs earlier scope, was UC14+)

| Column | Type | Required | Notes |
|--------|------|----------|-------|
| `id` | UUID | ✓ | PK only today |

**Correction (owner, 21/07):** Equipment/Machine is **many-to-many with
Process** (a process can involve several machines; a machine can be
isolated to see which process(es) reference it) — same shape as the
Activity correction above, not a simple `process_id` FK. This also likely
needs its own UC (registering a machine, linking it to a process, a
graphical view per machine) rather than being folded into UC2 — `architect`
+ `database-integrations` design the schema; `uc-scaffolder` drafts the UC
through the esteira gate when the owner is ready to formalize it (currently
being explored in Figma first).

**Also needs a 3D asset reference, not just name/label** — engineers link
real SolidWorks-exported 3D models (glTF/STL) to a Machine; see
[architecture.md § 3D prototype visualization](../../docs/architecture.md).
Whatever table holds this needs a file reference (storage URL + format),
not a plain text column — and it must be **nullable**, since the model is
optional per machine, not required.

### UC1 model

- `deviante.users` — credentials and session identity
- `deviante.managers` — personal profile: languages + optional location
- `deviante.processes` — manufacturing workflow: **company name**, name, description, sector

Register flow creates one row in `users` + `managers` in a single transaction.

## Portfolio schema (current)

Platform schema for alander.io — auth profiles, product registry, tracks, and
**UC/content rows** (SoT = live tables; vault sync is optional replica push —
see `partials/uc-esteira.md`).

| Table | Role |
|-------|------|
| `portfolio.users` | Owner/member roles; links to Supabase Auth UUID |
| `portfolio.products` | Product registry (`io`, `deviante`, `milebrick`, `harpia`); `show_in_apps` hides platform |
| `portfolio.product_access` | Which user may open which product app |
| `portfolio.access_requests` | Invitation / access request queue |
| `portfolio.artifacts` | Generic vault pointers (scope, diagrams) — legacy; prefer typed tables below |
| `portfolio.tracks` | SoundCloud playlist for `/tracks` |
| `portfolio.use_cases` | ABP use case — title, actor, object, pre/post, Why/What/Bounds, `body_md`, `metadata.esteira` |
| `portfolio.use_case_steps` | Steps table rows (step key, actor action, system response) |
| `portfolio.requirements` | Acceptance criteria → parent `use_cases` |
| `portfolio.use_case_relationships` | **Missing (owner, 21/07).** `<<include>>` (mandatory sub-flow) and `<<extend>>` (optional branch) between UCs currently live only as unsynced markdown inside `use_cases.body_md` ("Extension Points" / "Included Use Cases") — no structured column, no live SoT. Needs a real table (relationship type, source UC, target UC, note) so the DB knows a UC's include/extend links without a markdown intermediary. `architect` + `database-integrations` design this — same backlog item as the Activity many-to-many correction above. |
| `portfolio.publications` | Writing pieces — JSONB locales for title/excerpt/body |
| `portfolio.quests` | Roadmap quests per product/phase — public read, owner write; replaces static `content/gestalt-roadmap.json` |

**Not in v1:** no `portfolio.recruiters` or `portfolio.professionals` tables. **Recruiter** is a use-case actor (IO-UC6 guest via access key, not a DB table). Signed-in users use `portfolio.users.role` (`owner` | `member`) and `portfolio.product_access`.

DDL: `data/schema/portfolio/*.sql`. Apply order: [data/schema/portfolio/README.md](../../data/schema/portfolio/README.md).

**Web client:** Supabase Dashboard → **Project Settings → API → Exposed schemas** must include `portfolio` (and `deviante`, `milebrick`, `harpia` for product apps). Without this, PostgREST returns **406** and the portfolio site shows empty use cases/publications even when Auth login works. Seed owner rows: `data/seed/portfolio/owner_users.sql`.

Public RLS: anon reads published/public rows; owner manages all. See `grants.sql`.

## Milebrick schema (current)

| Table | Role |
|-------|------|
| `milebrick.languages` | Lookup: BCP 47 code + display name |
| `milebrick.users` | Account; `ui_language_code` FK |
| `milebrick.native_languages` | Profile: languages the user speaks (0..N) |
| `milebrick.target_languages` | Profile: languages the user learns (0..N), CEFR + `is_active` |
| `milebrick.contexts`, `lexical_units`, `practices`, … | Content (language column → FK in a later pass) |

**Removed:** `learners`, `user_languages` — replaced by `native_languages` + `target_languages`.

### Attribute reference

#### `milebrick.languages`

| Column | Type | Notes |
|--------|------|-------|
| `code` | VARCHAR(10) PK | BCP 47 lowercase (`pt`, `en`) |
| `name` | VARCHAR(100) | Display label |

#### `milebrick.users`

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID PK | |
| `name` | TEXT | |
| `email` | TEXT | Unique |
| `roles` | TEXT | Comma-separated roles |
| `ui_language_code` | VARCHAR(10) FK | App interface language |
| `created_at` | TIMESTAMPTZ | |

#### `milebrick.native_languages`

| Column | Type | Notes |
|--------|------|-------|
| `user_id` | UUID FK | → `users` |
| `language_code` | VARCHAR(10) FK | → `languages` |
| PK | `(user_id, language_code)` | |

#### `milebrick.target_languages`

| Column | Type | Notes |
|--------|------|-------|
| `user_id` | UUID FK | → `users` |
| `language_code` | VARCHAR(10) FK | → `languages` |
| `proficiency` | VARCHAR(2) | CEFR: A1–C2 |
| `is_active` | BOOLEAN | Current learning focus; at most one `true` per user |
| `created_at` | TIMESTAMPTZ | |
| PK | `(user_id, language_code)` | |

Partial unique index: one `is_active = true` row per `user_id`.

### Migrating legacy `milebrick` (16 tables)

Historical note: older dumps may predate the `milebrick` schema name.
**Do not** refer to Flashbrix as a product. Out of active scope — see
[partials/active-scope.md](../../partials/active-scope.md).

**Live state (2026-07-14):** all 16 tables are id-only UUID scaffolds (`gen_random_uuid()`). No FKs. `languages` has no `code` column.

| Script | Purpose |
|--------|---------|
| `000_inspect_milebrick.sql` | Audit columns + FKs (run if unsure) |
| `001_migrate_profile_languages.sql` | `languages(code,name)`, extend `users`, profile tables |
| `002_seed_user_languages.sql` | Dev user + native/target languages |

**Still id-only after 001:** `contexts`, `practices`, `flashbricks`, `vocabulary_blocks`, … — extend when those features ship.

## Ad-hoc read queries without MCP

If the Supabase MCP connector is unavailable (not linked, or blocked by an
Anthropic-side auth error unrelated to this repo), **do not stop at
"blocked."** An established direct-connection pattern already exists —
scripts named `scripts/diag-*.mjs` (e.g. `diag-owner-uc.mjs`,
`diag-uc-visibility.mjs`) connect straight to the live Supabase Postgres
using `node-postgres` (`pg`) with credentials parsed from
`deviante/api/.env` (`DATABASE_JDBC_URL`, `DATABASE_USER`,
`DATABASE_PASSWORD`). No `pg` install exists at repo root — `require` it via
`createRequire('c:/gestalt/deviante/web/package.json')`, the one workspace
that already has it in `node_modules`.

Use this pattern for **read-only** ad-hoc queries (esteira status, schema
inspection, sanity checks) any time MCP is down. It is *not* a replacement
for the owner-confirmation gate — writes to `portfolio.use_cases` (marking
`spec_confirmed`, etc.) still require the owner's explicit yes in-session;
see [uc-esteira.md](../../partials/uc-esteira.md). This is a read fallback,
not a permission bypass.

## Workflow when changing schema

1. Edit the `.sql` file in `data/schema/{product}/`
2. Run the script in **DataGrip** against the shared DB (ALTER or recreate in dev)
3. Update API repositories/queries with `{schema}.` prefix
4. Do **not** add migration folders under product repos (`deviante/api/`, etc.)

## API persistence (Deviante)

Ktor connects to the same PostgreSQL database. Persistence is layered:

```
Routing (HTTP)
    → Service (UC rules, BCrypt, transactions)
        → Repository (SQL via Exposed)
            → deviante.* tables
```

| Layer | Path | Role |
|-------|------|------|
| Config | `application.yaml` | JDBC URL, credentials |
| Pool | `Database.kt` | HikariCP + Exposed `Database.connect` |
| Table | `db/UsersTable.kt` | Maps `deviante.users` columns |
| Repository | `repository/UserRepository.kt` | `findByEmail`, `insert`, `existsByEmail` |
| Service | `service/AuthService.kt` | *(next)* register/login using `users` + `managers` |

**Phase 1 (current):** `UserRepository` on `deviante.users` only — insert and lookup by email.

**Phase 2:** `ManagerRepository` + register transaction (users + managers).

**Phase 3:** Auth routes (`/api/auth/register`, `/api/auth/login`) + JWT.

Exposed maps the qualified table name explicitly:

```kotlin
object UsersTable : Table(name = "users") {
    override val tableName = "deviante.users"
    // columns...
}
```

### Connection config (Supabase)

```yaml
# deviante/api/src/main/resources/application.yaml
database:
  jdbcUrl: ${DATABASE_JDBC_URL:jdbc:postgresql://localhost:5432/postgres?sslmode=require}
  user: ${DATABASE_USER:postgres}
  password: ${DATABASE_PASSWORD:}
```

Set env before `.\gradlew.bat run` (PowerShell):

```powershell
$env:DATABASE_JDBC_URL = "jdbc:postgresql://db.<ref>.supabase.co:5432/postgres?sslmode=require"
$env:DATABASE_PASSWORD = "<database-password-from-supabase>"
```

Use the **database password**, not `service_role` or `anon` keys, for JDBC.

### Web (Supabase Auth)

```env
# deviante/web/.env
VITE_SUPABASE_URL=https://<ref>.supabase.co
VITE_SUPABASE_ANON_KEY=<anon-key-from-supabase-settings-api>
```

Dashboard: Project Settings → API (`Project URL`, `anon` `public` key).

## API connection

Supabase Postgres (same DB DataGrip uses):

```
jdbc:postgresql://db.<ref>.supabase.co:5432/postgres?sslmode=require
```

Prefer explicit `deviante.table` in SQL/ORM — do not rely on `search_path` alone.

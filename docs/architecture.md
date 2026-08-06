---
title: Gestalt system architecture
owner: Alander (design@alander.io)
updated on: 21/07/2026
---

# Gestalt — system architecture

**This file is the unique source of truth for system architecture** and for
the **hub folder plant** (repo layout blueprint).  
Agent `architect` owns updates. Agent `truth-keeper` and `declutter` treat
replicas (chat, sprint notes, ad-hoc READMEs) as non-authoritative if they
disagree with this document — including live folders that diverge from the
plant.

Related: [gestalt-database](../skills/gestalt-database/reference.md),
[design-system/README.md](../../design-system/README.md),
[partials/active-scope.md](../partials/active-scope.md),
[partials/sot-matrix.md](../partials/sot-matrix.md),
[partials/kit-navigation.md](../partials/kit-navigation.md),
[partials/uc-esteira.md](../partials/uc-esteira.md). Tooling: agnostic kit + situational hosts — [ai-tooling.md](../partials/ai-tooling.md) (`maestro`).

## Tooling: knowledge is **tool-agnostic** (`gestalt-kit/`); current hosts/limits live in `partials/ai-tooling.md` — `maestro` conducts with both.

## Active products (scope)

Only **Portfolio (IO)** and **Deviante (DV)** are in active scope.  
Milebrick and Harpia are **out of scope**. The name **Flashbrix** is forbidden
(legacy alias — must never appear as a product). See
[partials/active-scope.md](../partials/active-scope.md).

**Applied schema SoT** is the **live Supabase** database. Bootstrap DDL:
`data/schema/ensure_active.sql` (idempotent). After objects exist live,
`declutter` may remove matching files under `data/schema/` with owner yes.

**Use-case intent + runtime SoT** is live `portfolio.use_cases` (+ steps,
requirements). Vault markdown under `gestalt-kit/vault/` is an Obsidian
authoring replica — default on drift: **DB wins**. Owner confirmation
before coding: [partials/uc-esteira.md](../partials/uc-esteira.md).

**Components SoT** is Figma ADS; product screens are styled instances
(`ui-designer`) — see sot-matrix.

## Layered stack (every feature)

```
ORCA Hub (objects / CTAs / roles)     ← vocabulary (ooux)
        ↓
Schema: data/schema/{product}/*.sql   ← applied on Supabase Postgres
        ↓
API persistence: Exposed Table + Repository + JDBC (Hikari)
        ↓
HTTP: Ktor routes (JSON)
        ↓
Web: React/Vite (Supabase Auth; business data via API and/or intentional supabase-js)
        ↓
Deploy: per registered remote (polyrepo-shipper)
```

Design delivery (visual):

```
Figma ADS (raw components, no product theme)
        ↓
Product library + styles (themed instances)
        ↓
tokens/ → product web UI
```

Agent `ui-designer` owns product styling / contrast / visual application.
ADS remains the **raw** component source; product screens must be **styled**.

**Figma-style direct manipulation (recurring product-feel principle,
owner 21/07):** objects get created instantly with a default (e.g.
"Untitled" process, UC2) or appear in place on a canvas the Manager
directly manipulates (the process-mining graph, UC4) — no modal-heavy
"fill a form, then click Save" pattern where it can be avoided. Edits
persist as they happen. `ui-designer` / `ux-engineer` apply this whenever a
new Deviante screen involves object creation or canvas interaction.

## Process mining exception (UC4 Upload Event Log onward)

CSV/XES → visual process trace is **process mining computation**, not a
generic CRUD business rule — PM4Py (Python) has no practical JVM
equivalent. A separate **Python + FastAPI** service performs this specific
computation; the Ktor/Kotlin API calls it and persists the **result**
(traces, operations) through the normal Exposed/Repository pattern.

**Kotlin still owns persistence end to end** (see [system-requirements.md
§ SR2](../partials/system-requirements.md)) — the Python service is
stateless compute, not a second place business data lives or a second
source of truth. Design this service boundary (`architect` +
`database-integrations`) as part of the same schema-design step already
scheduled before UC4 implementation — do not let it sprawl into other UCs
(UC2 Maintain Process, UC3 Maintain Activity are plain CRUD, no Python
involved).

## Defined process vs observed process

Deviante keeps two process representations deliberately separate:

- **Defined model:** the global Activities selected for a Process through
  `deviante.process_activities`. It exists before any Event Log and is plain
  Kotlin-owned CRUD.
- **Observed process:** the directly-follows graph returned by `/graph`,
  derived only from mapped events and traces in the latest valid Event Log.

The defined model does not store canvas coordinates. Its future ordering and
relations will use a semantic graph contract after the Figma interaction is
validated; automatic layout belongs to the frontend visualization. Until that
contract exists, do not infer edges from insertion order.

Mapping should prioritize Activities already selected for the Process, then
offer the rest of the global catalog. A defined Activity absent from a log
remains reusable in the defined model but is excluded from the observed graph
and has no frequency, duration, or drift-analysis parameters.

**Visualization reference (owner 21/07, PIBITI/CNPQ evidence grounding):**
the graph rendered after upload is a Disco-style directly-follows graph —
nodes are activities, edges are frequency- and duration-weighted paths,
with density sliders (Activities / Paths) to simplify the view and a
detail panel per node (frequency + performance stats). This follows van
der Aalst's process mining methodology, the academic reference for the
thesis evidence — not an arbitrary chart choice.

## Drift detection + predictive maintenance dependency

`deviante/Adaptive-Detection-of-Performance-Related-Temporal-Drifts-main/`
is Luiz Picolo's master's-thesis codebase (river/ADWIN + PM4Py) — the
concrete implementation behind IPDD drift detection (UC12) and
**predictive failure analysis**: percentage-based windowing that
expresses the probability of a given machine failing, a capability
beyond currently-scoped UCs.

The stewardship boundaries are:

- **Alander:** sole Deviante product owner, developer, and publisher; PIBITI
  researcher responsible for turning the research into a cohesive live
  product.
- **Eduardo Loures:** PIBITI advisor, PUCPR Mechatronics professor, research
  partner, and qualified tester.
- **Luiz Picolo:** master's researcher working at Bosch, research partner,
  qualified tester, and source of the vendored ADWIN/dataset implementation.
- **IPDD provenance:** the underlying IPDD research originates in the work of
  Denise Sato and Edson Ruschel. Product integration must retain this
  scientific provenance.

Eduardo and Luiz advise and test; they do not own Deviante delivery or publish
the product. Agents likewise assist Alander and must not be represented as
human delivery owners.

Architecturally this is the same stateless-compute-service boundary as
the Process mining exception above: the FastAPI service wraps
ADWIN/prediction in addition to PM4Py parsing; Kotlin still owns
persistence end to end. Domain detail (actors, entity notes):
[deviante-domain § Predictive maintenance
dependency](../skills/deviante-domain/reference.md).

**For `architect` (explicit, owner 21/07):** the actionables on the live
site (drift alerts, failure-probability, maintenance recommendations)
connect to Luiz's algorithms **through this FastAPI service as a thin
adapter, not a fork.** Luiz's scripts (`adwin_dataset.py`,
`adwin_streaming.py`, `adwin_real_dataset.py`, `prediction_final.py`) stay
as close to untouched as possible — the service imports/calls them (or
factors them into importable functions with minimal surgery), it does not
rewrite his detection/prediction logic in Kotlin or reimplement it by
hand. Preserving the integrity and provenance of the research code is a hard
constraint here, not a style preference. When `architect` +
`database-integrations` design this
service boundary (already scheduled before UC4/UC12), treat "wrap, don't
rewrite" as a design constraint on the same footing as the persistence
boundary above.

**Access:** researcher/tester labels describe collaboration context, not
product ownership. Runtime permissions remain intentionally simple for the
current happy path: Alander uses `owner`; Eduardo and Luiz use `mentor`.
Mentors can view and edit shared Processes but cannot delete them. Repository
publication and production deployment remain Alander's responsibility outside
the product role model.

**Housekeeping (not done here):** the vendored folder is double-nested
(`Adaptive-Detection-of-Performance-Related-Temporal-Drifts-main/Adaptive-Detection-of-Performance-Related-Temporal-Drifts-main/`)
— looks like a plain zip-extract artifact; `declutter` can flatten it
with owner yes.

## 3D prototype visualization (Machine linking, owner 21/07)

**The 3D model is optional per Machine.** Some equipment will have one,
some won't — a Machine with no model attached must still work normally
(show a neutral placeholder, not an error or a blocked state). When
`uc-scaffolder` drafts the Machine UC, this needs an explicit AC: absence
of a 3D model is a valid, first-class state, not an incomplete one.

Engineers link **real 3D CAD prototypes** to a Machine — models built in
SolidWorks, not a flat icon or illustration. SolidWorks' native format
(`.sldprt`/`.sldasm`) is not web-renderable; the realistic pipeline is
**export → web-friendly 3D format (glTF/GLB or STL) → upload → render**
with a browser 3D viewer (e.g. `<model-viewer>` or three.js). This is an
open technical question, not yet decided:

- Where the exported file lives (Supabase Storage bucket vs. an external
  asset host) and what `deviante.equipment` (or a related asset table)
  needs to reference it.
- Which format(s) to accept/require from the engineer (glTF preferred —
  smaller, PBR-ready; STL as a fallback with no color/material data).
- Whether conversion (SolidWorks → glTF) happens outside Deviante (engineer
  exports before upload) or Deviante ever needs to convert server-side —
  default assumption: **outside**, engineer exports before upload, until
  the owner says otherwise.

`architect` + `database-integrations` design this alongside the
Machine↔Process many-to-many correction — same backlog item, don't split
it into a third parallel discussion.

## Endorsed patterns

| Area | Pattern | Location |
|------|---------|----------|
| API persistence | `*Table` + `*Repository` + transaction | `deviante/api/.../db`, `repository` |
| DB pool | HikariCP + Exposed `Database.connect` | `Database.kt`, `application.yaml` |
| SQL ownership | DDL in `data/schema/{product}/`; always `schema.table` | apply in DataGrip — no Flyway in API |
| Auth | Supabase Auth on web; JDBC for business tables | [seed-accounts](../skills/seed-accounts/reference.md) |
| UC → code | ABP first → API → web | [implement-deviante-uc](../skills/implement-deviante-uc/reference.md) |
| Quests | `portfolio.quests` runtime | [product-progress](../skills/product-progress/reference.md) |
| Kit ↔ site CMS | **Kit↔DB reconciliation** (poll-based Observer) | § below + `scripts/check-kit-drift.mjs` |
| Deviante stack boundary | Kotlin/Exposed (OOP) owns **all** Deviante persistence + business logic. `deviante/web` (TS/React) only calls the Ktor API — it never talks to Postgres directly or reimplements business rules in JS/TS. **Portfolio is the sole exception** (may use `supabase-js` directly, per Auth row above). `deviante-backend` must verify a Kotlin persistence layer exists before wiring a route, and create it (following `UsersTable`/`UserRepository`) if it doesn't — not fake it in JS. | `deviante/api/**` vs `deviante/web/**`; [active-scope.md](../partials/active-scope.md) |

**Rule:** prefer extending a pattern above over inventing a parallel one.
If you propose a new pattern, document it **here** (this file) before
spreading it across the codebase.

## Kit navigation (tool-agnostic bootstrap)

**SoT:** [partials/kit-navigation.md](../partials/kit-navigation.md) (declutter +
truth-keeper enforce; maestro teaches). Cold-start skill: `kit-entry`.

Every host loads the same `gestalt-kit/` tree. IDE folders are **adapters only**:

| Host | Adapter | Regenerate |
|------|---------|------------|
| Cursor | `.cursor/skills/<name>/SKILL.md` | `node scripts/sync-cursor-adapters.mjs` |
| Claude Code | `--plugin-dir gestalt-kit` | `.claude/skills/README.md` |

Do **not** author Gestalt skills under Cursor global `skills-cursor/` or
parallel `claude-*` / `cursor-*` skill names in the kit.

## Kit↔DB reconciliation (poll-based Observer)

**Decision (architect, 2026-07-20):** do **not** use classic GoF **Observer**
(push: Subject notifies subscribers on every write). We have no shared event
bus between git files and Supabase, and push via Postgres `NOTIFY` / webhooks
is overkill for ~40 kit rows.

**Endorse instead:** a **poll-based Observer / drift watchdog** — same idea
as `truth-keeper` (“re-check every time”), applied to kit content:

| Store | Role |
|-------|------|
| `gestalt-kit/` | **Authoring** SoT (agents edit files here) |
| Live `portfolio.kit_docs` | **Runtime / site** SoT (owner edits on `/kit` persist here) |

Both are legitimate **update** surfaces. Neither Cursor adapters, Claude
worktrees, Obsidian notes, nor stub `doc/` / `gestalt-vault/` may hold a
third authoring copy. The owner also edits via **DataGrip** — same depara
rules as `/kit` Save. Contract: [partials/kit-depara.md](../partials/kit-depara.md).

**When to observe (dev):**

1. Session open (`maestro` kit check) — `node scripts/check-kit-drift.mjs --depara`
   when env or MCP available
2. After editing kit files or after Save on alander.io `/kit`
3. Before `sync-kit.mjs` (know direction: kit→DB vs protect site bodies)

**Mechanism:** compare `(kind, slug)` sets + `md5(body)` (local files vs
`select kind, slug, md5(body_md) … from portfolio.kit_docs`). Prefer
**Supabase MCP** for the DB side (no long dumps).

| Verdict | Meaning | Typical repair |
|---------|---------|----------------|
| `aligned` | hashes match | none |
| `kit_ahead` / body longer in files | files newer | `sync-kit.mjs` **only if owner says repo wins** |
| `db_ahead` / body differs after site Save | site/DataGrip newer | **Default:** keep DB; depara → optional git writeback if owner wants repo replica |
| `drift` | mix / orphans | owner picks direction per slug |

**Not endorsed (yet):** push Observer (DB trigger → agent), automatic
two-way writeback site→git. Add only with owner yes + SoT edit here.

## Design system — Deviante dark/red theme

**Decision (architect, 2026-07-21):** Deviante's web UI adopts the
dark-background / red-accent visual language prototyped in the Figma
export **Process Mining Canvas Design**
(`ZZKdwxgmeCNJFG64zGbADe`), superseding the prior light-blue palette.
The dark/red language now applies to the authenticated app surfaces
(login, dashboard, canvas); the standalone public landing page was
removed on 2026-08-05 — `/` redirects straight to `/login`.

| Layer | Source of truth |
|-------|------------------|
| Tokens | `deviante/web/src/index.css` (`--background #0d1017`, `--primary #dc2626`, `--accent-strong #991b1b`, Inter + JetBrains Mono) |
| Components | Tailwind v4 + shadcn/Radix, copied under `deviante/web/src/components/ui/*.jsx` (lower-case filenames: `button.jsx`, `card.jsx`, …) |
| Legacy bespoke components | `Alert.jsx` / `Button.jsx` renamed to `LegacyAlert.jsx` / `LegacyButton.jsx` — still used by pages not yet migrated (`LoginPage`, `AccountSettingsPage`, `AuthCallbackPage`, `Modal`); migrate them to the shadcn set incrementally, do not add new bespoke CSS classes |
| Global base fixes needed for the Tailwind layer to behave (added 2026-07-21) | `@layer base` around anchor color reset (cascade-layer precedence bug); `* { border-color: var(--border) }` (shadcn convention Tailwind v4 expects) |

**Not yet done:** the canonical Figma file `DzMGsKozRhijjcFFngdy4S` (Deviante's
own design file) does not reflect this palette yet — `ui-designer` owns
updating it so Figma stops drifting from shipped code.

### Onde os exports do Figma Make moram (decisão do owner, 2026-07-22)

Exports do Figma Make ficam em **[`figma-make/`](../../figma-make/README.md)**
na raiz do hub, e **só o README é versionado** — o conteúdo é gitignored.
O owner larga o export ali, a sessão porta pro produto, o owner apaga o que
já foi implementado. Sumir é o estado normal.

Duas regras que não são cosméticas:

- **Nada em `deviante/web` ou `portfolio` pode `import` de dentro de
  `figma-make/`.** O código porta o design; não depende do export. Se a
  pasta sumir, o build tem que continuar passando.
- **Comentário de arquivo portado cita o arquivo Figma + a versão**, não um
  caminho dentro de `figma-make/` (que é transiente).

**Por que a regra existe:** em 21/07 havia duas cópias do "Process Mining
Canvas Design" — uma de 1802 linhas na raiz do hub e uma de 2374 (Version
20) dentro de `deviante/web/`. O port seguiu a antiga, e por isso a
`ProcessDetailPage` virou uma rota-formulário separada em vez do que o
design manda. Corrigido em 22/07: a tela de processo é uma só
([ProcessCanvasPage.jsx](../../deviante/web/src/pages/ProcessCanvasPage.jsx)),
com o grafo e o formulário como abas — `ProcessDetailPage` e
`ProcessMiningPreviewPage` foram removidas.

**Endorsed instead of:** the light-blue palette and the bespoke
`.home-*` / `.card` CSS-class system for new work — extend the shadcn
component set in `components/ui/` rather than adding parallel hand-rolled
classes.

## Anti-patterns

- Second auth password store in app tables for login
- Flyway / auto-migrate inside `deviante/api`
- Disabled / "coming soon" UI (see [partials/owner-preferences.md](../partials/owner-preferences.md))
- New markdown files when an existing doc should be updated
  ([prefer-existing-files](../skills/prefer-existing-files/reference.md))
- Treating Milebrick, Harpia, or Flashbrix as active products
- Sprint nicknames treated as architecture or quest SoT
- Maintaining a second copy of skills/agents under stub `doc/`,
  `.claude/worktrees/**`, or Obsidian notes — **gestalt-kit + live DB only**
- Treating IDE adapters (`.cursor/skills` thin links) as editable SoT
- Recreating a second vault at `gestalt-vault/` or fat docs under `doc/`
  (plant: vault = `gestalt-kit/vault/` only)

## Folder plant (hub layout SoT)

**Architect owns this plant.** `declutter` and `truth-keeper` verify live
folders against it. Obsidian opens **`gestalt-kit/vault/`** only (not the
whole kit).

```
gestalt-kit/                 ← unique knowledge home in the hub
├── README.md
├── agents/
├── skills/
├── partials/
├── docs/
│   └── architecture.md      ← this file (architecture + plant SoT)
├── vault/                   ← Obsidian root (UCs, MOCs, writing)
│   ├── .obsidian/
│   ├── Gestalt.md
│   ├── io/
│   ├── products/
│   └── writing/
└── plans/                   ← sprint + UC template
    ├── sprint-plan-2026-07.md
    └── UC-Template.md
```

**Sem stubs (decisão do owner, 22/07/2026):** `doc/`, `gestalt-vault/` e
`infra/` foram removidos por inteiro. Antes, `doc/README.md` e
`gestalt-vault/README.md` existiam como redirects — não existem mais.

**Não recriar:** `doc/**` (incluindo `doc/agents/`, `doc/products/`,
`doc/.obsidian/`) e `gestalt-vault/**`. Docs antigos ainda citam esses
caminhos; o alvo real está em `gestalt-kit/`. As regras de `.gitignore`
para `infra/cloudflared/credentials.json` e `config.yml` foram mantidas
de propósito — se `infra/` voltar, os segredos continuam protegidos.

**Kit CMS sync** indexes only `agents/`, `skills/`, `partials/`,
`docs/architecture.md` — never `vault/` or `plans/`.

**Out of plant (product repos, later):** PIBITI report under `deviante/docs/`
— pointer only in kit README until a later move.

## Change control

1. Propose the architectural change (architect).
2. Owner approves.
3. Update **this file** first (including folder plant when layout changes).
4. Then update replicas (code, skills, agents).

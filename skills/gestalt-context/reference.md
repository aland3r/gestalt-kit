# Gestalt Context

## What is Gestalt?

Gestalt is an umbrella workspace containing four products under the **Alander Brands and Products (ABP)** naming scheme. **Knowledge** (agents, skills, vault prose, plans) lives under **`gestalt-kit/`**. Obsidian opens **`gestalt-kit/vault/`** only.

## Products

| Code | Name | Folder | Remote | Priority |
|------|------|--------|--------|----------|
| IO | Portfolio | `portfolio/` | github.com/aland3r/portfolio | Active |
| DV | Deviante | `deviante/api/`, `deviante/web/` | github.com/aland3r/deviante-api, deviante-web | Active (PIBITI) |

**Out of scope:** Milebrick (MB), Harpia (HA). **Forbidden name:** Flashbrix
(legacy — never a separate product). See
**[partials/active-scope.md](../../partials/active-scope.md)**.

Architecture + folder plant SoT: **[architecture.md](../../docs/architecture.md)** (agent `architect`).

## Deviante (DV)

Decision-support system for industrial maintenance management. Uses machine learning and process mining to detect equipment failures before breakdown.

- **Actor:** Manager (industrial maintenance decision-maker)
- **API:** Kotlin/Ktor — `deviante/api/`
- **Web:** React/Vite — `deviante/web/`
- **PIBITI:** Brazilian undergraduate research program; code repos are deliverable artifacts

## Portfolio (IO)

Frontend-only personal portfolio deployed via GitHub Pages.

- **Folder:** `portfolio/`
- **Stack:** Next.js
- **Deploy:** `.github/workflows/deploy.yml`

## Owner Preferences

See **[partials/owner-preferences.md](../../partials/owner-preferences.md)** — "no clutter" rule + examples.  
See **[partials/active-scope.md](../../partials/active-scope.md)** — IO+DV only; no Flashbrix.  
Prefer updating existing files: **[prefer-existing-files.md](../prefer-existing-files/reference.md)**.  
Consistency / no junk: **[repo-consistency.md](../repo-consistency/reference.md)**.

Kept as shared partials/skills on purpose; link them instead of restating.

## Workspace Conventions

- Open Cursor at `c:\gestalt` (umbrella), not individual product repos
- Git remote for umbrella: `aland3r/gestalt-hub` (kit, data, tokens — **not** product code; enables Cursor Remote Control)
- Knowledge home: `gestalt-kit/` (agents, skills, vault, plans)
- Navigation contract: [partials/kit-navigation.md](../../partials/kit-navigation.md)
- Cursor adapters: `.cursor/skills/` (thin links — `node scripts/sync-cursor-adapters.mjs`)
- Use cases: **SoT = live** `portfolio.use_cases` (site `/cases`, MCP); vault
  markdown under `gestalt-kit/vault/` is an authoring replica — see
  [use-cases-surface.md](../use-cases-surface/reference.md) and
  [uc-esteira.md](../../partials/uc-esteira.md) (owner gate before coding)
- Publications: vault `writing/` ↔ `portfolio.publications`

- Stubs only: `doc/README.md`, `gestalt-vault/README.md` (redirects)
- PIBITI report (Deviante): `deviante/docs/relatorio/` (not moved to kit yet)
- Database DDL: `data/schema/{product}/` — one PostgreSQL DB, schema-qualified queries (`deviante.users`). See [database.md](../gestalt-database/reference.md).

## Folder Map

```
gestalt/
├── gestalt-kit/          Unique knowledge home
│   ├── agents/ skills/ partials/ docs/
│   ├── vault/            Obsidian (UCs, publications, MOCs)
│   └── plans/            Sprint + UC template
├── doc/                  Stub redirect → gestalt-kit/
├── gestalt-vault/        Stub redirect → gestalt-kit/vault/
├── deviante/
│   ├── api/
│   ├── web/
│   └── docs/relatorio/   PIBITI report (separate from vault UCs)
├── portfolio/            Next.js site (github.com/aland3r/portfolio)
├── data/                 Shared PostgreSQL DDL
├── scripts/sync-vault.mjs
├── ui/dev-quest/
└── tokens/
```

Historical folders (`milebrick/`, `harpia/`) may still exist on disk but are
**out of scope** — do not extend them.

## When Working Here

1. Read the product MOC in `gestalt-kit/vault/` (`io/Portfolio.md`, `products/deviante/Deviante.md`, …)
2. For Deviante features: load the UC from live DB (esteira gate) before coding;
   vault files are replicas — [uc-esteira.md](../../partials/uc-esteira.md)
3. For Portfolio platform features, read the UC in `gestalt-kit/vault/io/user stories/` before coding
4. Match existing patterns in the target product folder
5. Keep documentation in English

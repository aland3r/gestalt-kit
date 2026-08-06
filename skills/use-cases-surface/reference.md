# Use cases surface

Public use-case CMS for alander.io — inspired by [Google Antigravity](https://antigravity.google) “Use cases” nav. Every Gestalt product exposes specs; **IO** is the platform layer, not a fourth app.

## Architecture

```
Live Supabase
portfolio.use_cases          ← SoT for UC intent + runtime
portfolio.use_case_steps
portfolio.requirements
portfolio.publications
       ▲
       │  owner edits on /cases  ·  MCP  ·  sync-vault (owner-directed)
       │
gestalt-kit/vault/           ← Obsidian authoring replica (not silent overwrite)
       │
       ▼  @supabase/supabase-js (anon + RLS)
alander.io /cases/*

gestalt-kit/            ← authoring (agents / skills / partials / architecture)
       │                  (sync-kit does NOT index vault/ or plans/)
       ▼  scripts/sync-kit.mjs
portfolio.kit_docs
       │
       ▼  same Supabase client + RLS
alander.io /kit
```

**Implementation gate:** [partials/uc-esteira.md](../../partials/uc-esteira.md)
— owner must confirm the live DB spec card before coding.

**User stories are structured rows in `portfolio.*` — not loose JSON or `portfolio.artifacts`.**
**Kit docs are structured rows in `portfolio.kit_docs` — site edits persist there (report surface); re-sync from kit overwrites body unless `--skip-existing-bodies`.**

| Table | Content |
|-------|---------|
| `use_cases` | IDs, title, actor, object, pre/post, Why/What/Bounds, `body_md`, `metadata.esteira` |
| `use_case_steps` | Step key + actor action + system response |
| `requirements` | Acceptance criteria (`DV-UC1-AC1`, …) |

| Layer | Role |
|-------|------|
| **Supabase** | **SoT** for UC intent/runtime; owner CRUD on `/cases` |
| **Vault** | Authoring replica + constellation graph via MOC wikilinks |
| **Git** | History for vault prose; does not beat live DB on drift |
| **JSON fallbacks** | `content/projects.json` etc. — retire after vault + sync stable |

## Product codes

| Code | Prefix | In `/apps` grid? |
|------|--------|------------------|
| `io` | ABP-IO | **No** — platform specs only |
| `deviante` | ABP-DV | Yes |
| `milebrick` | ABP-MB | Yes |
| `harpia` | ABP-HA | Yes (coming soon) |

Query apps with `portfolio.products WHERE show_in_apps = true`.

## Visibility & RLS

| Table | Public read | Owner |
|-------|-------------|-------|
| `use_cases` | `visibility = 'public'` AND `status IN ('ready','shipped')` | full |
| `use_case_steps` | when parent UC is public | full |
| `requirements` | when parent UC is public | full |
| `publications` | `status = 'published'` | full |

Owner preview of drafts: authenticated + `portfolio.is_owner()`.

## Public routes (portfolio web)

| Route | Content |
|-------|---------|
| `/use-cases` | Hub — IO platform + product cards |
| `/use-cases/io` | Platform UCs (ABP-IO) |
| `/use-cases/deviante` | Product UC list |
| `/use-cases/{product}/{slug}` | UC detail (markdown body + requirements) |

Do **not** reuse `/work` — that is career/experience timeline.

## Nav UX (Antigravity-style)

- **Products / Apps** dropdown — launchable products only
- **Use cases** dropdown — hub + per-product links
- **Download CV** pill (top-right) — ABP-IO-UC1, not buried in utilities menu

## Authoring

- Vault paths: see [vault/README.md](../../vault/README.md)
- UC template: [write-use-case.md](../write-use-case/reference.md) — paths under `gestalt-kit/vault/`
- Esteira / owner gate: [uc-esteira.md](../../partials/uc-esteira.md)
- After editing vault: `node scripts/sync-vault.mjs` only with owner direction
  when reconciling toward DB (default on drift: **DB wins**)

## Publications

Prefer markdown in `gestalt-kit/vault/writing/` over `content/projects.json` + i18n JSON. JSON existed for static GitHub Pages export; markdown + JSONB locales in `portfolio.publications` is the long-term shape.

## Related

- [partials/uc-esteira.md](../../partials/uc-esteira.md) — owner confirmation gate
- [database.md](../gestalt-database/reference.md) — `portfolio.*` DDL
- [product-progress.md](../product-progress/reference.md) — roadmap quests vs public UC surface
- [gestalt-context.md](../gestalt-context/reference.md) — IO vs product apps

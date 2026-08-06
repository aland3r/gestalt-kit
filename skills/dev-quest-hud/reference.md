# Dev Quest HUD (arcade dev mode)

Gestalt-wide **development progress UI** for React/Vite apps: Quest Log, page stamp, arcade loading. **Dev builds only** — never shipped to production users.

Canonical package: **`ui/dev-quest/`**

## When to use

- Adding or wiring dev-quest HUD to any Gestalt product web app
- Creating `{product}/web/src/lib/roadmap.js` for a new product
- Replacing plain "Carregando…" with `ArcadeLoadingScreen`

## Quick setup

1. Read **[ui/dev-quest/README.md](../../ui/dev-quest/README.md)**
2. Add Vite alias `@gestalt/dev-quest` → `ui/dev-quest`
3. Import `@gestalt/dev-quest/dev-quest.css`
4. Wrap app in `DevQuestProvider` with product-specific `phases` + `loadingLines`
5. Render `DevQuestHud` once at app root; `DevQuestStamp` in footers; `ArcadeLoadingScreen` on loading states

## Roadmap data

Each product owns **`src/lib/roadmap.js`** (quests + status). Long-form plan lives in product docs (e.g. `deviante/docs/roadmap.md`).

### Quest ID format (all apps)

```
UC{use-case}-{group}{letter}     e.g. UC1-1a, UC1-1b, UC1-2a
0.x                              cross-cutting infra (DDL, env)
```

| UC1 group | Meaning |
|-----------|---------|
| **UC1-1*** | **Authentication first** — Supabase Auth, session, guards, OAuth |
| **UC1-2*** | Profile for existing users |
| **UC1-3*** | Sign-up UX — **out of v1 dev** (not in quest log) |

Full convention: **[roadmap-granularity.md](../roadmap-granularity/reference.md)**

Quest status values: `'done'` | `'active'` | `'locked'`.

## Rules

- **Do not** show Quest Log in production (`isDevQuestEnabled()` gates everything)
- **Do not** duplicate components per product — import from `@gestalt/dev-quest`
- Keep loading lines on-brand per product (manufacturing vs language learning vs …)
- Update roadmap doc + `roadmap.js` together when phases change

## Related

- [seed-accounts.md](../seed-accounts/reference.md) — auth model
- [gestalt-context.md](../gestalt-context/reference.md) — no clutter

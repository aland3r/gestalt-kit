# Roadmap granularity (Gestalt)

All Gestalt products use the same **quest ID** pattern in `{product}/web/src/lib/roadmap.js` and product docs (`{product}/docs/roadmap.md`).

## Quest ID format

```
UC{use-case}-{group}{letter}
```

| Part | Meaning | Example |
|------|---------|---------|
| `UC1` | Use case from ABP spec | UC1 Maintain account |
| `-1` | Theme group within the UC | `1` = auth/session, `2` = profile, `3` = sign-up UX |
| `a`, `b`, `c` | Ordered step in that group | UC1-1a before UC1-1b |

**Cross-cutting infra** (not tied to one UC): numeric `0.1`, `0.2`, … in phase P0.

**Status:** `done` | `active` | `locked` — one `active` quest at a time per product.

## UC1 groups (all apps)

| Group | Scope | Typical steps |
|-------|--------|----------------|
| **UC1-1** | **Authentication** (first — always) | Supabase env → client → login → session → logout → guards → Google OAuth |
| **UC1-2** | **Profile** (existing users) | Load profile → edit account settings → OAuth language gate (if needed) |
| **UC1-3** | **Sign-up UX** | **Out of v1 dev** — no register page; users created in Supabase dashboard |

Auth is **not** out of scope — only **self-service account creation UI** is.

## Per-product differences

Same UC1-1* steps everywhere; profile table differs:

| Product | UC1-2 profile table |
|---------|---------------------|
| Deviante | `deviante.managers` |
| Milebrick | `milebrick.users` + language tables |
| Harpia | TBD |
| Portfolio | TBD |

## When to split a new letter

Add `UC1-1d` when the previous step is **shippable and testable** on its own (e.g. login works before OAuth).

## Related

- [dev-quest-hud.md](../dev-quest-hud/reference.md) — Quest Log UI
- [seed-accounts.md](../seed-accounts/reference.md) — Supabase Auth + dashboard users
- [database.md](../gestalt-database/reference.md) — Supabase Postgres + schemas

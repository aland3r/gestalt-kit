# Product progress (Gestalt)

Track and display **development progress** per product from scope, use cases, and quests.

## Model

| Layer | Location | Purpose |
|-------|----------|---------|
| **Scope** | `{product}/docs/scope.md` | v1.0 boundaries — which UCs ship |
| **Use cases** | `gestalt-kit/vault/` | ABP specs + acceptance criteria → sync to Supabase |
| **Roadmap (human)** | `{product}/docs/roadmap.md` | Phases, quest table, active step |
| **Roadmap (machine)** | See below | Drives % on UI |

### Machine-readable roadmap

| Product | Live data | Public UI |
|---------|-----------|-----------|
| **Portfolio (IO)** | `portfolio.quests` (Supabase, since 20/07/2026) — `content/gestalt-roadmap.json` is fallback only | `BuildProgress` (`/cases`) + `GamifierHud` (floating, site-wide) on alander.io |
| **Deviante** | `portfolio.quests` (Supabase, since 20/07/2026) — `deviante/web/src/lib/roadmap.js` is offline fallback only | `GamifierHud` (floating, site-wide, public) on deviante.alander.io, plus `DevQuestHud` in dev builds |
| **Milebrick, Harpia, …** | `{product}/web/src/lib/roadmap.js` | Dev Quest HUD (dev builds only) — not yet moved to Supabase |

Portfolio and Deviante both show progress publicly now — see [gamifier.md](../gamifier/reference.md) for the widget and the UC→quest auto-sync. Milebrick/Harpia keep `@gestalt/dev-quest` for local dev until they're wired the same way.

### Machine-readable roadmap

| Layer | Location | Purpose |
|-------|----------|---------|
| **Live source of truth** | `portfolio.quests` (Supabase; `product_code` covers IO/DV/MB/HA) | `RoadmapProvider` fetches on every Portfolio page load; public read, owner write |
| **Fallback / bootstrap** | `portfolio/content/gestalt-roadmap.json` | Only when the DB table is empty/unreachable; do not keep parallel `quests_seed.sql` — live `portfolio.quests` is SoT |
| **Portfolio (IO) slice** | `portfolio/content/roadmap.json` | IO-only legacy static file; no longer read by the live site |
| **Deviante, Milebrick, …** | `{product}/web/src/lib/roadmap.js` | Dev Quest HUD (dev builds); still file-based, not migrated to Supabase |

**Editing a quest status now means editing the `portfolio.quests` row** (Supabase Table Editor, a SQL `UPDATE`, or the `/ship-quest` command in `gestalt-kit/`) — editing the JSON file alone no longer changes the live site.

Portfolio `/cases` shows **quest %** and **UC bars** filterable by product (IO, DV, MB, HA) — **public read for all visitors** (the gamified progress view); only creating/editing a UC still requires owner sign-in. Products in **designing** show status badge instead of % when they have no quests.

## Alternative paths in scope

Scopes must register **primary**, **alternate**, and **out-of-scope** paths (see Deviante UC1: auth `UC1-1*` vs sign-up UX `UC1-3` deferred). Do not document only the happy path.

## Product lifecycle

| Status | UI label | Quest log |
|--------|----------|-----------|
| `designing` | Designing | Empty — ORCA / scope phase |
| `developing` | Developing | Active quests |

Set `status` in `portfolio.products` (Supabase) and mirror in `gestalt-roadmap.json` + `products.js` until UC8 reads DB only.

## Quest rules

- ID format: `UC{n}-{group}{letter}` — see [roadmap-granularity.md](../roadmap-granularity/reference.md)
- Status: `done` | `active` | `locked` — **one** `active` quest per product
- Each quest must map to a UC in `scope.md`
- Add `"UC{n}-1x … documented"` quest when the use case spec is written

## Progress metrics

| Metric | Formula | Best for |
|--------|---------|----------|
| **Quest completion** (default) | `done / total` quests | Day-to-day dev; matches Quest Log |
| **UC coverage** | per-UC `done / total` quests | Stakeholder view by feature |
| **Acceptance criteria** | AC checked / total AC | Fine-grained QA (manual checklist) |
| **Phase gate** | weight phases (e.g. LAUNCH = 40%) | When late phases matter more |

**Recommendation:** ship **quest %** publicly; use **UC bars** in detailed view; reserve **AC %** for release candidates (manual or test map).

**Portfolio complete / version label:** umbrella completion (all four products' UCs shipped + owner v1 approval) and the `GESTALT v0.xx` headline are defined in [partials/portfolio-completion.md](../../partials/portfolio-completion.md) — do not infer from quest % alone.

## When to update

1. Change **scope.md** when adding/removing a UC from v1.0
2. Write **user story** before marking a doc quest `done`
3. Set quest `done` when step is shippable and verified
4. Set next quest to `active`; sync `roadmap.md` table
5. Portfolio: sync `gestalt-roadmap.json` when IO or mirrored DV quests change; site rebuild picks up JSON

## Agent checklist

- [ ] New feature has a UC row in `scope.md`
- [ ] Use case file exists (`ABP-{PRODUCT}-UC{n}-…`)
- [ ] Quests added to JSON/JS roadmap
- [ ] `roadmap.md` phase table updated
- [ ] Only one `active` quest

## Related

- [roadmap-granularity.md](../roadmap-granularity/reference.md)
- [dev-quest-hud.md](../dev-quest-hud/reference.md)
- [gamifier.md](../gamifier/reference.md) — public widget + UC→quest auto-sync
- [write-use-case.md](../write-use-case/reference.md)

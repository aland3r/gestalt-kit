# Gamifier

Owns the **public, always-on quest log widget** (`GamifierHud`, `ui/gamifier/`) and the machinery that keeps it fed without hand-editing: turning UCs into quests, and syncing quest status as UCs change. Complements [product-progress.md](../product-progress/reference.md) (the broader progress model) and [dev-quest-hud.md](../dev-quest-hud/reference.md) (the dev-only sibling this widget is modeled on — **never merge the two**, one is a debug tool, the other ships to production on purpose).

## Responsibilities

1. **Inform progress** — `GamifierHud` (`ui/gamifier/GamifierHud.jsx`) is a floating widget, mounted unconditionally (no dev-only gate), on:
   - Portfolio (`alander.io`): `FloatingGamifierHud` inside `ClientRoot.jsx`'s `FloatingDock`, fed by the existing `RoadmapProvider`/`useRoadmap()` context.
   - Deviante (`deviante.alander.io`): mounted in `App.jsx`, fed by `useGamifierProducts()` (`deviante/web/src/lib/gamifier.js`).
   - Both read `portfolio.quests` (Supabase) — Deviante's quests are no longer file-only; `deviante/web/src/lib/roadmap.js` is now an **offline fallback**, not the primary source.
2. **Transform UCs into quests** — every row in `portfolio.use_cases` gets exactly one linked "spec" quest automatically (`quest_id = 'uc{n}-spec'`, `auto_synced = true`, `use_case_id` set). This happens two ways:
   - **Trigger** (`use_cases_create_quest` in `quests_alter.sql`) — fires on every future UC insert.
   - **Backfill** (`quests_uc_sync_backfill.sql`) — ran once to link/create the "spec" quest for UCs that already existed (linked the pre-existing `"UC{n} documented"` rows for IO/Milebrick/Harpia; inserted new ones for Deviante, which had none).
3. **Persist as UCs are edited** — the `use_cases_sync_quest` trigger (`quests_alter.sql`) fires `AFTER UPDATE OF status ON portfolio.use_cases` and flips the linked `auto_synced` quest to `done`/`locked` to match. No app code and no manual SQL involved.

## After UC accepted (esteira)

When a UC finishes the [owner gate](../../partials/uc-esteira.md) and
implementation verifies against confirmed ACs:

1. Prefer MCP. If MCP is missing, **ask** before claiming quests/UC rows
   changed.
2. Set `portfolio.use_cases.metadata.esteira.review_status` → `accepted`
   (and keep or set lifecycle `status` to `ready`/`shipped` when the owner
   wants the public/spec quest flipped).
3. **Align fine-grained quests to valid UC scope** — list
   `portfolio.quests` for that `use_case_id` / `uc_number`. Mark done only
   steps that match the **confirmed** ACs and shipped work; unlock/activate
   the next in-scope quest; leave out-of-scope rows `locked` (or ask owner
   to drop/relabel — do not invent scope).
4. Do **not** hand-edit `-spec` quests (`auto_synced`) — flip via UC
   `status` instead.
5. Optionally run `/ship-quest` for individual implementation quests.

This is the hook that keeps the public HUD honest to what the owner actually
confirmed — not to a sprint nickname or a stale vault file.


## What is still manual — and why

Only the **one "spec" quest per UC** is automatic. The fine-grained
implementation-subtask quests already seeded (e.g. `UC1-1a`..`UC1-1f`) stay
on `/ship-quest` **plus** the post-accept sync above. The `use_cases` row
does not observe “is the code shipped” beyond lifecycle `status` and
`metadata.esteira`. Do not auto-derive subtask status from heuristics.

## File map

| Concern | File |
|---|---|
| Schema: link column + triggers | `data/schema/portfolio/quests_alter.sql` |
| One-time backfill | Already applied on live DB — no seed file; use SQL editor only for gaps |
| Shared fetch (Vite + Next, via `@gestalt/auth`) | `ui/auth/quests.js` |
| Widget (presentational, no fetching) | `ui/gamifier/GamifierHud.jsx`, `ui/gamifier/gamifier.css` |
| Row→UI shaping helper | `ui/gamifier/roadmap-shape.js` (`buildGamifierProducts`) |
| Portfolio wiring | `portfolio/app/components/FloatingGamifierHud.jsx`, mounted in `ClientRoot.jsx` |
| Deviante wiring | `deviante/web/src/lib/gamifier.js` (`useGamifierProducts`), mounted in `App.jsx` |

## Persistent version legend (0.xx → 1.0, added 21/07/2026)

Full contract (umbrella = IO+DV+MB+HA, portfolio-complete query, approval gate):
[partials/portfolio-completion.md](../../partials/portfolio-completion.md).

`GamifierHud`'s header shows `GESTALT {version}` next to the eyebrow —
e.g. `GESTALT v0.52`. **The number is never computed client-side** — it
reads the trigger-maintained singleton row `portfolio.gestalt_version`
(`data/schema/portfolio/gestalt_version.sql`) via `fetchGestaltVersion()`
(`ui/auth/quests.js` / `portfolio/lib/gestalt-auth/quests.js`), which every
`GamifierHud` copy takes as a prop (`gestaltVersion`) and formats with
`formatVersionLabel()`.

**How the number moves:**

- A Postgres trigger (`recompute_gestalt_version()`, fires on
  `portfolio.quests` insert/delete/status-update and on
  `portfolio.products.metadata` update) recomputes and upserts the row on
  every relevant change — no app code, no cron, no manual recompute step.
- Formula: `done / total` quests across the **umbrella** (`io`, `deviante`,
  `milebrick`, `harpia` — [portfolio-completion.md](../../partials/portfolio-completion.md)),
  rounded to 2 decimals, **capped at 0.99** until portfolio-complete **and** all
  four products have `v1_approved_at`.
- `1.0` only when **every** umbrella product has
  `portfolio.products.metadata->>'v1_approved_at'` set **and** every umbrella UC
  is `shipped` + `public` + `esteira.review_status = accepted` — manual
  per-product approval plus DB portfolio-complete check; never automatic:

  ```sql
  UPDATE portfolio.products
  SET metadata = metadata || jsonb_build_object(
    'v1_approved_at', now(), 'v1_approved_by', '<who approved>'
  )
  WHERE code IN ('io', 'deviante', 'milebrick', 'harpia');
  ```

  That `UPDATE` itself fires the trigger, so `gestalt_version` flips to
  `1.0` immediately — no separate step needed.
- If `gestalt_version` hasn't loaded yet (first paint, or the table
  predates a given deploy), `formatVersionLabel()` falls back to a coarse
  `pré-v1`/`v1` label derived from `products[].v1ApprovedAt` — never the
  primary source, just a paint-before-fetch placeholder.

No agent should set `v1_approved_at` automatically, ever — not even when
every quest shows `done`. Duplicated by hand in
`ui/gamifier/GamifierHud.jsx` (canonical), `portfolio/app/components/GamifierHud.jsx`,
and Deviante's vendored copy — same copy-by-hand caveat as the rest of this
file (`useGamifierProducts()` now returns `{ products, gestaltVersion }`,
not a bare array — update both destructuring sites if you touch it).

## Rules

- `GamifierHud` never gates on `isDevQuestEnabled()` — that check belongs only to `@gestalt/dev-quest`. If a future product genuinely needs to hide progress (e.g. still in `designing` lifecycle with zero quests), handle it by passing an empty `products` array, not by reintroducing a dev-only flag here.
- Every new Vite product wired through `gestaltDevQuest()` (`ui/dev-quest/vite-gestalt.js`) already gets the `@gestalt/gamifier` alias for free — add the widget the same way Deviante did (a small provider hook + one mount in `App.jsx`), do not duplicate `GamifierHud`/`gamifier.css` per product.
- Portfolio and Deviante currently resolve `@gestalt/auth` to **two different directories** (`portfolio/lib/gestalt-auth/` vs `ui/auth/`) — when adding a new shared fetch function, add it to `ui/auth/` first (that's what Vite apps get) and port it into `portfolio/lib/gestalt-auth/` too if portfolio needs it, same as `quests.js` was done. They are not one package yet; don't assume changing one changes the other.
- **Deviante has a THIRD, vendored copy** at `deviante/web/vendor/gestalt/` (gitignored, `README.md`: "so CI builds without the monorepo checkout"). `deviante/web/vite.config.js` hardcodes its `gestaltDevQuest` import from this vendored path (`./vendor/gestalt/ui/dev-quest/vite-gestalt.js`), **regardless of whether you're running inside the monorepo** — editing `ui/dev-quest/vite-gestalt.js` (or `ui/auth/`, `ui/gamifier/`) at the monorepo root has **zero effect** on Deviante's dev server or production build until the matching files are copied into `deviante/web/vendor/gestalt/` too. There is no sync script; this is copy-by-hand, same as everything else in this repo. Forgetting this step is the most likely way a future gamifier change "works everywhere except Deviante."
- When adding a product's quests to Supabase for the first time (e.g. Milebrick/Harpia moving past `designing`), run `quests_uc_sync_backfill.sql`'s pattern again for that product code, or simply insert UCs into `use_cases` and let `use_cases_create_quest` do it.

## Related

- [partials/uc-esteira.md](../../partials/uc-esteira.md) — owner gate; gamifier runs after accept
- [product-progress.md](../product-progress/reference.md) — broader progress model, quest ID format, lifecycle
- [dev-quest-hud.md](../dev-quest-hud/reference.md) — the dev-only sibling widget
- [database.md](../gestalt-database/reference.md) — schema conventions, apply order
- [ship-quest](../ship-quest/SKILL.md) — command to close one fine-grained quest

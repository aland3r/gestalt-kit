<!--
PARTIAL: kit↔Supabase depara contract — truth-keeper + command /kit-depara.
Both gestalt-kit/ (files) and portfolio.kit_docs (live) are legitimate update
surfaces. DataGrip SQL and /kit Save both write the DB side.
-->

# Kit↔Supabase depara

**Scope:** `gestalt-kit/` indexed docs ↔ live `portfolio.kit_docs` only.  
**Not in scope:** `vault/`, `plans/`, `use_cases`, `content_strings`, schema DDL
— each has its own SoT row in [sot-matrix.md](sot-matrix.md).

## Two legitimate surfaces

| Surface | Who updates | Typical tool |
|---------|-------------|--------------|
| **Authoring** | Agent / owner in git | Edit `gestalt-kit/agents|skills|partials|docs/` |
| **Runtime** | Owner on live DB | Portfolio `/kit` Save · **DataGrip** on Supabase · rare service SQL |

**Neither surface is automatically wrong when they differ.** Depara reports
*what* diverged and *likely provenance*; the owner picks direction per slug
(or batch).

DataGrip is the owner's normal mediation path for Supabase — treat manual
`UPDATE portfolio.kit_docs …` the same as a `/kit` Save for depara purposes.

## SoT policy (owner — kit_docs runtime)

**Baseline (2026-07-21):** repo (`gestalt-kit/`) was synced to
`portfolio.kit_docs` (48 rows, `aligned`). From that point:

| Phase | Who wins | When |
|-------|----------|------|
| **Default (now)** | **Live DB** (`portfolio.kit_docs`) | Owner edits on `/kit` or **DataGrip** to save tokens — no agent rewrite required |
| **After manual DB/UI edit** | Still **DB** until owner runs depara | Owner invokes `depara kit supabase` when they want agents to see live text or reconcile git |
| **Exception: bulk kit ship** | Owner says **repo wins** | One-shot `sync-kit.mjs` (e.g. after many agent edits in git) — then DB wins again |

**Token economy:** prefer editing `/kit` or DataGrip for copy tweaks; run
depara + truth-keeper when agents must read current runtime text — cheaper
than pasting bodies in chat. `maestro` may suggest this route.

**Agents must not** overwrite `portfolio.kit_docs` from git unless the owner
explicitly says **repo wins** for that batch.

## Owner command (truth-keeper)

Invoke **`truth-keeper`** with any of:

- `depara kit supabase`
- `depara kit↔db`
- `/kit-depara`
- `kit depara`

The agent **must run or request** a live diff before recommending writes.

## Mechanism (read-only first)

1. **Script (preferred):**
   ```bash
   node scripts/check-kit-drift.mjs --depara
   ```
   Uses Postgres when `DATABASE_USER` + `DATABASE_PASSWORD` are set, else
   `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY`, else prints MCP SQL.

2. **Supabase MCP** (when script env missing):
   ```sql
   select kind, slug, md5(body_md) as body_md5, length(body_md) as body_len,
          source_path, updated_at
   from portfolio.kit_docs
   order by kind, slug;
   ```
   Compare to local inventory from the script without `--depara` (stdout table).

3. **Never** claim `aligned` without comparing hashes to live rows.

## Verdicts per slug

| Verdict | Meaning | Likely cause |
|---------|---------|--------------|
| `aligned` | Same `md5(body)` | — |
| `kit_ahead` | Row missing in DB or file mtime newer | Git edit not synced |
| `db_ahead` | Row missing in kit index or `updated_at` newer | `/kit` Save or **DataGrip** |
| `body_drift` | Same key, different hash | Both sides edited — **ask owner** |

## Repair options (owner yes required)

| Situation | Option A | Option B |
|-----------|----------|----------|
| `kit_ahead` | `node scripts/sync-kit.mjs` (kit→DB) | — |
| `db_ahead` (site/DataGrip) | Write DB body back to git file at `source_path` | Keep DB as runtime truth; document in truth-keeper owner notes |
| `body_drift` | Owner names winner per slug | Split: sync some slugs, writeback others |
| `db_only` (orphan row) | Delete row in DataGrip / SQL | Restore file in kit + keep row |
| `kit_only` (new file) | `sync-kit.mjs` insert | — |

**Protect site edits when bulk syncing:**  
`node scripts/sync-kit.mjs --skip-existing-bodies` — skips overwriting bodies
that already exist in DB (use when unsure which side wins).

## truth-keeper output shape (depara)

```text
Depara: gestalt-kit ↔ portfolio.kit_docs
Live access: ok | MCP only | blocked
Verdict: aligned | kit_ahead | db_ahead | body_drift | drift

Aligned (n): …
Kit ahead (n): kind/slug — file mtime … — → sync-kit.mjs
DB ahead (n): kind/slug — updated_at … — likely /kit or DataGrip — ASK: writeback to git?
Body drift (n): kind/slug — kit_len … db_len … — ASK: which SoT for this slug?
DB only (n): …
Kit only (n): …

Ask (if any db_ahead / body_drift): For each slug, keep DB or git?
Next (after owner answers): …
```

## Boundaries

- Depara is **read-only** until the owner confirms repair direction.
- Do not run `sync-kit.mjs` or git writeback without explicit yes.
- Schema / grants changes in DataGrip are **DDL domain** — separate check vs
  `data/schema/` and `ensure_active.sql`, not this depara.

# kit-depara — procedure

Load agent **`truth-keeper`** mindset; do not implement product code.

## 1. Confirm scope

Only **`gestalt-kit/` → `portfolio.kit_docs`**.  
If the owner meant UCs, schema, or `content_strings`, say so and switch SoT
row from [sot-matrix.md](../../partials/sot-matrix.md).

## 2. Run live diff

```bash
node scripts/check-kit-drift.mjs --depara
```

If env is missing, paste MCP SQL from [kit-depara.md](../../partials/kit-depara.md)
and diff manually against local inventory:

```bash
node scripts/check-kit-drift.mjs
```

Env (any one):

| Vars | Access |
|------|--------|
| `DATABASE_USER` + `DATABASE_PASSWORD` | Postgres pooler (DataGrip path) |
| `SUPABASE_URL` + `SUPABASE_SERVICE_ROLE_KEY` | Supabase JS |
| Neither | MCP `execute_sql` only |

## 3. Classify each mismatch

Use file mtime vs `updated_at` as **hint only** — owner may have edited both.

- **DB newer** → likely `/kit` Save or **DataGrip** mediation.
- **File newer** → likely git/agent edit not synced.

## 4. Report using depara output shape

See partial § truth-keeper output shape. Always list slugs needing owner
decision (`db_ahead`, `body_drift`, orphans).

## 5. Wait for owner direction

Per slug or batch:

| Owner says | Action |
|------------|--------|
| "git wins" / "sync kit" | `node scripts/sync-kit.mjs` (optional `--dry-run` first) |
| "DB wins" / "keep site" | Write `body_md` to `source_path` in git, or document exception |
| "protect site bodies" | `sync-kit.mjs --skip-existing-bodies` |
| "delete orphan" | SQL in DataGrip with owner yes |

## 6. Re-run depara

After repair, `node scripts/check-kit-drift.mjs --depara` must trend to
`aligned` (or documented intentional exceptions in truth-keeper owner notes).

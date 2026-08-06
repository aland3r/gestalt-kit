<!--
PARTIAL: Gestalt portfolio completion — progress %, version label, UC coverage.
Link from: product-progress, gamifier, truth-keeper, maestro, sot-matrix,
active-scope, product-manager, kit-navigation.
Do not copy-paste — link this file.
-->

# Portfolio completion — progress & version

**Contract:** headline **progress** and **version** on alander.io and product
sites must reflect **live Supabase**, not hand-edited JSON, chat memory, or
client-side guesses. The **Gestalt portfolio** (umbrella: IO + DV + MB + HA)
is **complete** only when **every use case** of **every product** in that set
is concluded in the DB.

This partial defines *measurement* and *completion*. It does **not** override
[active-scope.md](active-scope.md): agents still **build** IO + DV only until
the owner expands scope — MB/HA count toward completion even while stubbed.

## Products in the umbrella

| `product_code` | ABP prefix | Name |
|----------------|------------|------|
| `io` | ABP-IO | Portfolio (alander.io) |
| `deviante` | ABP-DV | Deviante |
| `milebrick` | ABP-MB | Milebrick |
| `harpia` | ABP-HA | Harpia |

Never treat Flashbrix as a fifth product — see [active-scope.md](active-scope.md).

## When is the portfolio complete?

**Done** = for each of the four `product_code` values above, **every**
non-deprecated row in `portfolio.use_cases` has:

| Field | Required value | Notes |
|-------|----------------|-------|
| `status` | `shipped` | Lifecycle terminus for that UC |
| `metadata.esteira.review_status` | `accepted` | Owner verified ACs on the esteira ([uc-esteira.md](uc-esteira.md)) |
| `visibility` | `public` | Anonymous `/cases` surface ([use-cases-surface](../skills/use-cases-surface/reference.md)) |

**Portfolio complete** when the grouped counts below show `shipped = total` for
**all four** products (and no product has `total = 0` unless owner explicitly
retired that product from the umbrella).

Diagnostic (read-only):

```sql
SELECT product_code,
       count(*) FILTER (WHERE status = 'shipped') AS shipped,
       count(*) FILTER (WHERE status NOT IN ('deprecated')) AS total,
       count(*) FILTER (
         WHERE status = 'shipped'
           AND visibility = 'public'
           AND metadata->'esteira'->>'review_status' = 'accepted'
       ) AS portfolio_complete_ucs
FROM portfolio.use_cases
WHERE product_code IN ('io', 'deviante', 'milebrick', 'harpia')
GROUP BY 1
ORDER BY 1;
```

Agents must **not** declare “Gestalt v1 shipped” from quest % alone.

## Three progress layers (do not mix)

| Layer | SoT table | UI | Purpose |
|-------|-----------|-----|---------|
| **Headline version** | `portfolio.gestalt_version` (singleton) | `GamifierHud` eyebrow `GESTALT v0.xx` | Gradual **0.00 → 1.00** label |
| **Quest progress** | `portfolio.quests` | Quest log %, phase bars | Day-to-day dev granularity |
| **UC coverage** | `portfolio.use_cases` | `/cases` UC list + per-UC bars | Stakeholder view by feature |

**Rule:** widgets read DB rows; they do not re-derive the headline number from
local arrays except a **paint-before-fetch** fallback documented in
[gamifier](../skills/gamifier/reference.md).

## Version numbering (`portfolio.gestalt_version`)

**Target contract** (this partial):

1. **Numerator / denominator** — all quests (or, equivalently, all UCs once
   quest sync is complete) for **`io`, `deviante`, `milebrick`, `harpia`**.
2. **`version`** — `round(done / total, 2)`, capped at **`0.99`** while any
   product lacks `portfolio.products.metadata->>'v1_approved_at'`.
3. **`1.00`** — only when **both**:
   - portfolio-complete query above is satisfied for all four products, **and**
   - every row in `portfolio.products` for those codes has
     `metadata.v1_approved_at` set (manual owner + manager sign-off — never
     automatic).

Implemented in `data/schema/portfolio/gestalt_version.sql` — trigger
`recompute_gestalt_version()` on `portfolio.quests`, `portfolio.products`,
and `portfolio.use_cases`.

Owner sets approval (per product, fires trigger → `1.00` when all approved):

```sql
UPDATE portfolio.products
SET metadata = metadata || jsonb_build_object(
  'v1_approved_at', now(),
  'v1_approved_by', '<who approved>'
)
WHERE code = '<io|deviante|milebrick|harpia>';
```

## Keeping DB layers consistent

| Event | DB effect | Agent duty |
|-------|-----------|------------|
| UC inserted | `use_cases_create_quest` → spec quest row | Ensure UC exists in vault/scope before insert |
| UC `status` → `ready` / `shipped` | `sync_quest_from_use_case` marks auto-synced quests `done` | Run gamifier / `/ship-quest` for non-auto quests |
| Quest `status` updated | `recompute_gestalt_version` refreshes singleton | Use `/ship-quest`; never edit JSON alone |
| UC published on `/cases` | `visibility` + `status` on `use_cases` | Match [uc-esteira](uc-esteira.md) gate |
| Product approved for v1 | `products.metadata` update | Owner-only; never agent-initiated |

**Drift checks** (`truth-keeper`):

- `use_cases.status = shipped` but quests for that UC not `done` → report
- `gestalt_version.done_quests` ≠ live quest count for umbrella products → report
- Vault markdown ≠ `portfolio.use_cases` → [uc-esteira](uc-esteira.md) (DB wins)

## UI surfaces (must stay aligned)

| Surface | Progress source | Version source |
|---------|-----------------|----------------|
| Floating `GamifierHud` (IO, DV sites) | `portfolio.quests` via `fetchAllQuests` | `fetchGestaltVersion()` → `gestalt_version` |
| `/cases` `BuildProgress` | quests + `use_cases` merge | N/A (product filter IO/DV/MB/HA) |
| Owner beacon menu | writes `use_cases.status/visibility` | indirect via triggers |

Shared fetch lives in `ui/auth/quests.js` and is mirrored in
`portfolio/lib/gestalt-auth/quests.js`. Deviante also vendors a copy under
`deviante/web/vendor/gestalt/` — sync by hand when changing fetch logic
([gamifier](../skills/gamifier/reference.md)).

## Agent checklist

- [ ] Claiming “X% done” → cite live `portfolio.quests` or `gestalt_version` query
- [ ] Claiming “UC N done” → cite `portfolio.use_cases` row (`status`, esteira)
- [ ] Claiming “portfolio complete” → run portfolio-complete SQL above
- [ ] Changing version formula → edit `gestalt_version.sql`, apply on Supabase, not React
- [ ] MB/HA work → measure in completion; build only when [active-scope](active-scope.md) expands

## Related

- [active-scope.md](active-scope.md) — what agents build now (IO + DV)
- [uc-esteira.md](uc-esteira.md) — owner gate before ship
- [sot-matrix.md](sot-matrix.md) — quest + UC SoT rows
- [product-progress](../skills/product-progress/reference.md) — quest ID format, lifecycle
- [gamifier](../skills/gamifier/reference.md) — widget wiring + vendored copies

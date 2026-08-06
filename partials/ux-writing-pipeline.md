<!--
PARTIAL: content production handoff — strategist brief → ux-writer → surfaces.
Link from: content-strategist, ux-writer, maestro.
-->

# UX writing pipeline

**Strategy** (`content-strategist`) decides *what* and *why*. **Production**
(`ux-writer`) turns an approved brief into *ready strings* for UI, i18n, kit,
and vault — without inventing a second strategy.

## Flow

```
content-strategist → Strategy card (owner yes)
        ↓
ux-writer → Copy pack (strings + placement + char notes)
        ↓
researcher / ux-engineer / ui-designer → fit check (on demand)
        ↓
Owner or implementer → portfolio i18n, /kit Save, DataGrip, code
```

## Copy pack must include

| Field | Purpose |
|-------|---------|
| `Surface` | Route / component / i18n prefix |
| `Strings` | Final text per key or label |
| `ORCA / terminology` | Nouns used — must match Hub + [terminology](../skills/terminology/reference.md) |
| `Constraints` | Max length, Carbonot chrome vs Planar prose ([portfolio-typography.md](portfolio-typography.md)) |
| `Persona lens` | Which vault persona was checked (via `researcher` if run) |
| `Do not ship` | Strings rejected or deferred |

## Surfaces (IO active)

- `portfolio/content/i18n/*.json` — site chrome + pages
- `portfolio.kit_docs` / `/kit` — kit titles, summaries, body (DB wins runtime)
- `portfolio.use_cases` — titles, descriptions, steps (UC esteira gate)
- `gestalt-kit/vault/writing/` — long-form when strategist assigns channel

## Boundaries

- `ux-writer` does not publish strategy or ORCA workshops — `content-strategist` / `ooux`.
- `ux-writer` does not commit without owner yes on the pack.
- Runtime kit text: DB wins after baseline — see [kit-depara.md](kit-depara.md).

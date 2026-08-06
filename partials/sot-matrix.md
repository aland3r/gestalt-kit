<!--
PARTIAL: SoT matrix — link from truth-keeper, declutter, architect, ui-designer.
-->

# Source-of-truth matrix (structures)

One domain → one SoT. Replicas follow; declutter deletes confirmed replicas.

| Domain | Unique SoT | Replicas (must match / disposable after sync) |
|--------|------------|-----------------------------------------------|
| **UI components (structure)** | Figma **ADS** library | Product Figma lib, `tokens/`, React under `{product}/web` — never invent a second component system |
| **Product visual theme** | Figma product **styles** + `tokens/` | React CSS — `ui-designer` applies theme; raw ADS alone is not enough on-product |
| **Portfolio typography** | `gestalt-kit/partials/portfolio-typography.md` | `portfolio/app/portfolio-tokens.css` implements; no third Carbonot size |
| **OOUX method** | Notion OOUX Masterclass DB | Agent facilitation notes |
| **OOUX vocabulary (per product)** | Notion ORCA Hub | UI copy, code nouns |
| **Use-case intent + runtime** | **Live Supabase** `portfolio.use_cases` (+ `use_case_steps`, `requirements`) | `gestalt-kit/vault/**/user stories/` (Obsidian authoring replica), site `/cases` — **DB wins** on drift; see `partials/uc-esteira.md` |
| **Quest progress** | Live Supabase `portfolio.quests` | Website HUD; local JSON = bootstrap only |
| **Gestalt version & portfolio completion** | Live `portfolio.gestalt_version` + `portfolio.use_cases` (+ triggers on `quests`, `products`) | `GamifierHud` version label; completion = all four products' UCs shipped — see `partials/portfolio-completion.md` |
| **Table shape (applied)** | **Live Supabase Postgres** | `data/schema/**` files — keep only until `ensure_active` / inventory proves applied; then declutter may delete |
| **DDL bootstrap (not yet applied)** | `data/schema/ensure_active.sql` + per-table files for gaps | Do not invent parallel migration folders in product APIs |
| **Agents / skills (authoring)** | `gestalt-kit/` (`agents/`, `skills/`, `partials/`) | Thin adapters only (`.cursor/skills/`); **no** Claude/Obsidian full copies; stubs at `doc/` / `gestalt-vault/` are redirects only. Navigation contract: `partials/kit-navigation.md` |
| **Agents / skills (authoring + sync)** | `gestalt-kit/` (repo SoT) → `portfolio.kit_docs` (DB replica) | **Repo wins** on sync. Automated via `kit-sync-policy.md` (daily + change-triggered). `/kit` + DataGrip for token-free reads; edits go back to repo via git. `sync-kit.mjs` pushes local SoT to DB when owner approves or schedule fires |
| **Vault prose / Obsidian graph** | `gestalt-kit/vault/` (MOCs, writing, UC markdown replicas) | Must not silently overwrite live `portfolio.use_cases`; never store agent prompts in vault |
| **Hub folder plant (layout)** | `gestalt-kit/docs/architecture.md` § Folder plant | Live dirs on disk; declutter/truth-keeper verify against plant |
| **Sprint / UC template** | Latest `gestalt-kit/plans/sprint-plan-*.md` (+ `UC-Template.md`) | chat / empty `PLAN.md` — not quest SoT. `product-manager` always reads the live sprint file; `truth-keeper` names this path when plan SoT is disputed |
| **AI hosts / quotas** (situational) | `gestalt-kit/partials/ai-tooling.md` | Chat memory — `maestro` asks when stale; kit stays tool-agnostic |
| **MCP connectors authorized this session** (situational) | `gestalt-kit/partials/mcp-connectors.md` | Chat memory — tool UUIDs are per-session, only the registry's keywords are stable |
| **Kit navigation / parse contract** | `gestalt-kit/partials/kit-navigation.md` | `.cursor/skills/` thin adapters (`sync-cursor-adapters.mjs`); Claude `--plugin-dir gestalt-kit`; cold start via skill `kit-entry` |
| **System-wide requirements (SR-series)** | `gestalt-kit/partials/system-requirements.md` | — (index only; detail lives in the doc each SR points to) |

## Figma is always the design reference (owner, 2026-07-21)

**Governance:** design decisions belong to the **owner + design team**, not
to whichever agent is coding. An agent may *propose*; it does not decide
palette, component shape, or layout on its own read of a code export.

- **Figma files and Figma Make outputs are the reference** for any UI
  component or visual theme work — check the two rows above (ADS library,
  product styles) before writing CSS/components from scratch.
- **When Figma is unreachable or the answer isn't known** (no Figma MCP
  connected in this host, file access missing, node not found): **ask the
  owner** — do not freehand a design. The owner may also **decide in chat**
  to connect a Figma integration so components get registered **directly in
  the file**, rather than reverse-engineered from screenshots or code later.
- An explicit **owner decision stated in chat** is binding immediately (same
  "chat edit is binding" rule as [uc-esteira.md](uc-esteira.md)) — but it is
  a bridge, not a replacement: reconcile it into the actual Figma file as
  soon as that's possible, so Figma stays the durable SoT rather than chat
  history.
- **No frame yet?** Don't freehand it — see skill
  [figma-make-prompt](../skills/figma-make-prompt/reference.md): ground a
  Figma Make request in the real chrome + confirmed UC spec, let the owner
  run it, implement from what comes back.
- **Current Deviante authority (owner, updated 24/07):** the newest rotating
  Figma Make export under root `figma-make/` is the primary visual reference
  for the surface it covers. Copy that approved composition into
  `deviante/web` with high fidelity; do not couple production to the generated
  project. The export folder and ZIP are disposable inputs that the owner may
  replace or delete after the port. The durable result is the product code
  plus the canonical Figma file when reconciled.
- Figma's generated strings are proposals, not language SoT. `oouxer` owns
  object/CTA vocabulary, `ux-writer` owns final interface copy, `ux-engineer`
  owns hierarchy and fit, and `researcher` supplies evidence when needed.
  `ui-designer` preserves the visual intent while applying those decisions.
- The canonical **Deviante v1.0** Figma file
  (`DzMGsKozRhijjcFFngdy4S`) may lag behind the latest rotating Make export.
  That gap is design debt, not permission to freehand another visual system.

## Tool-agnostic vs situational

Knowledge structure does **not** fork per IDE. Current Cursor/Claude/Antigravity
limits **do** change how `maestro` routes work today — see `ai-tooling.md`.

## One knowledge home

**`gestalt-kit/`** is the only knowledge tree in the hub. Vault prose lives as
**`gestalt-kit/vault/`** (Obsidian opens that subfolder). Do not recreate
`doc/agents/`, `doc/products/`, or a second vault at repo root.

## Database vs `data/` folder

1. Run [`data/schema/ensure_active.sql`](../../../data/schema/ensure_active.sql) on Supabase (idempotent); policies via `data/schema/portfolio/grants.sql` if needed.
2. Inventory live tables (`list_tables` / SQL on `portfolio` + `deviante`).
3. **No `data/seed/`** — row data SoT is live DB. UC **intent/runtime** SoT is live `portfolio.use_cases`; vault markdown is an authoring replica (`scripts/sync-vault.mjs` only with owner direction when drift exists). Owner confirmation gate: `partials/uc-esteira.md`.
4. Out-of-scope schemas (`milebrick`, `harpia`, `flashbricks`) — do not expand; drop only with owner yes.

## Figma rule

If ADS already defines the component, code and product libraries **instance/theme** it — they do not redesign a parallel primitive. That is structure SoT, not “extra docs in data/”.

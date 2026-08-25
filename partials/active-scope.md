<!--
PARTIAL: link from skills/agents — do not copy-paste.
Referenced from: architecture.md, gestalt-context, repo-consistency,
truth-keeper, sprint-plan, prefer-existing-files (related).
-->

# Active product scope

**In scope (build, document, ship) — owner 24/08/2026 (supersedes 13/08):**

| Code | Name | Notes |
|------|------|--------|
| DV | Deviante | `deviante/` — **active again.** Developed **simultaneously** with Milebrick. PIBITI is delivered, but the product keeps evolving (new UCs, arc42 docs, LP). Thesis SoT in `gestalt-kit/docs/deviante/` stays **intocada** (delivered record, don't edit). |
| MB | Milebrick | `milebrick/web` + `milebrick/api` + `milebrick/doc` (inc. `milebrick/doc/ooux/`). Auth on Supabase (schema `milebrick`, **same project as portfolio**, invite-only / authorization-gated like Deviante). UC1 = auth. |
| IO | Portfolio | alander.io / `portfolio/` — parallel. |

**Owner works dynamically across all active products** — do not treat any of
DV/MB/IO as "frozen". The old priority-freeze registries (Deviante congelado,
Milebrick "só landing page / só prioridade") are **historical**; both DV and MB
are active build+doc+ship scope now. Deviante is **no longer exempt** from
`declutter` (an active product is subject to it again).

Harpia was **discontinued 18/08/2026** — the product no longer exists; its
folder, schema, DB rows and references were removed. Do not recreate it.

## Consistent product-site skeleton

Every active product's public site shares **one skeleton** — same tabs, same
shell — only the content differs per product. See
[lp-skeleton.md](lp-skeleton.md). The branded UI of each product **inherits**
from a single unbranded standard UI (tokens/attributes/parameters).

## Per-product architecture (arc42)

Each product documents its architecture as an **arc42** Markdown doc in its own
repo's `architecture/` folder (`deviante/docs/architecture/`,
`milebrick/doc/architecture/`), Obsidian-viewable, diagrams inline as
`mermaid`. The **hub** architecture stays in
[`../docs/architecture.md`](../docs/architecture.md). See
[arc42-structure.md](arc42-structure.md).

## Forbidden name: Flashbrix

**Flashbrix does not exist.** It was an old name for what became Milebrick.
Never create, restore, or mention Flashbrix as a separate product in vault,
Obsidian, docs, quests, or code comments. If you find a mention, delete or
rewrite it — do not "also document Flashbrix."

ABP codes: use `ABP-MB`, `ABP-IO`, `ABP-DV` for active work. Do not introduce
`ABP-FL` or Flashbrix paths under `doc/` or `gestalt-kit/vault/`. Milebrick is
the real product name; **Flashbrix stays forbidden** (above).

**Portfolio completion (measurement only):** the umbrella includes
**IO + DV + MB** — all UCs concluded in live DB defines when Gestalt is done.
See [portfolio-completion.md](portfolio-completion.md).

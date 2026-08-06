<!--
PARTIAL: link from skills/agents — do not copy-paste.
Referenced from: architecture.md, gestalt-context, repo-consistency,
truth-keeper, sprint-plan, prefer-existing-files (related).
-->

# Active product scope

**In scope (build, document, ship):**

| Code | Name | Notes |
|------|------|--------|
| IO | Portfolio | alander.io / `portfolio/` |
| DV | Deviante | PIBITI priority — `deviante/api`, `deviante/web` |

**Priority order (owner, 22/07/2026):** **1. Deviante**, then Portfolio.
Nothing else is built or changed until **every Deviante UC (UC1–UC15) is
functional**.

**Frozen — not out of the roadmap, just not now:**

| Code | Name | Notes |
|------|------|--------|
| MB | Milebrick | `milebrick/` — will be developed later. Do not delete, refactor, or document as active; do not touch anything inside, including the unversioned `milebrick/doc/ooux/` |
| HA | Harpia | `harpia/` — same freeze |

Both stay in the root `.gitignore` and are **exempt from `declutter`** — a
frozen product is not clutter.

## Forbidden name: Flashbrix

**Flashbrix does not exist.** It was an old name for what became Milebrick.
Never create, restore, or mention Flashbrix as a separate product in vault,
Obsidian, docs, quests, or code comments. If you find a mention, delete or
rewrite it — do not "also document Flashbrix."

ABP codes: use `ABP-IO`, `ABP-DV` only for active work. Do not introduce
`ABP-FL` or Flashbrix paths under `doc/` or `gestalt-kit/vault/`.

**Portfolio completion (measurement only):** the umbrella still includes
**IO + DV + MB + HA** — all UCs concluded in live DB defines when Gestalt is
done, even while MB/HA stay out of build scope here. See
[portfolio-completion.md](portfolio-completion.md).

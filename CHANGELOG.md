# Changelog

## 0.23.0 — 2026-08-04

- **`/debug-drift-detection` command** — evidence-first Deviante workflow from
  graph intent through explicit Process/Activity/Trace scope, Kotlin series
  construction, FastAPI IPDD/ADWIN, persistence, and reopen. Coordinates the
  existing maestro, truth-keeper, backend, frontend, architect, and database
  specialists; remains read-only unless the owner passes `--fix`.
- **Deviante analysis-scope invariant** — Process is the root context, while
  the statistical projection and Trace population are explicit, orthogonal
  inputs. A sole Activity can no longer be treated as semantically identical
  to whole-process duration merely because the exclusion list is empty.

## 0.22.0 — 2026-07-21

- **New Deviante dependency + actor:** `deviante/Adaptive-Detection-of-Performance-Related-Temporal-Drifts-main/`
  (Luiz Picolo's master's-thesis codebase) is the concrete IPDD/ADWIN
  drift-detection implementation behind UC12, and additionally ships
  percentage-based predictive failure analysis — capability beyond
  currently-scoped UCs. Luiz is Deviante's **mentor** stakeholder (new
  actor, distinct from Manager), invite-only and gated to
  `pafileiro@gmail.com`. Documented in `architecture.md § Drift detection
  + predictive maintenance dependency`, `deviante-domain/reference.md §
  Actors` + `§ Predictive maintenance dependency`, and sprint-plan
  `§ Modelo de acesso` / `§ Estado real do repositório`. Access itself is
  not yet granted — Luiz has not logged in yet (verified live, 21/07); no
  Deviante-side mentor permission tier exists in code — scope through
  `uc-scaffolder` + esteira before building one. Explicit constraint for
  `architect`: the FastAPI service wraps Luiz's ADWIN/prediction scripts
  as a thin adapter, not a fork/rewrite — preserving his codebase's
  integrity is a hard constraint, same footing as the persistence
  boundary.

## 0.21.0 — 2026-07-21

- **`skills/figma-make-prompt`** — token-economical path for a screen/UC
  with no Figma frame yet: ground a Figma Make request in the real,
  already-drawn chrome + confirmed UC spec, hand it to the owner to run,
  implement from what comes back — instead of freehanding pixels or
  speculative design+revise cycles. Serves the governance rule in
  `sot-matrix.md § Figma is always the design reference`. Worked example:
  the ABP-DV-UC2 Dashboard/Process Detail request (21/07).

## 0.20.0 — 2026-07-21

- **`partials/portfolio-completion.md`** — contract for headline progress,
  `gestalt_version`, and umbrella completion (all UCs shipped across IO, DV,
  MB, HA). Linked from sot-matrix, active-scope, kit-navigation,
  product-progress, gamifier, maestro, truth-keeper.

## 0.19.0 — 2026-07-21

- **`ux-writer`** — production copy from `content-strategist` briefs; pairs
  with researcher, ux-engineer, ui-designer. Pipeline:
  `partials/ux-writing-pipeline.md`.

## 0.18.0 — 2026-07-21

- **Renamed "spell" → "command"** across the kit (mental model, maestro's
  instrument table, `/kit` UI, `portfolio.kit_docs.kind`). Same meaning
  (skill with `disable-model-invocation: true`, owner-fired only) — the new
  name matches what these already are: real slash commands (`/ship-quest`,
  `/uc-gate`, `/kit-depara`), not a separate metaphor. Decided with `ooux`
  consulted on naming; historical changelog entries below keep "spell" as
  the term in use at the time.

## 0.17.0 — 2026-07-21

- **`product-manager`** — viability vs live sprint plan
  (`gestalt-kit/plans/sprint-plan-*.md`), UC esteira, quotas, optional
  reference models (MPS.BR / thin slice only). Anti foot-gun; does not
  invent a second plan SoT (`truth-keeper` maps the path).

## 0.16.1 — 2026-07-21

- **Kit runtime SoT** — after baseline repo→DB sync, **DB wins** for
  `portfolio.kit_docs`; owner edits `/kit` / DataGrip (token economy); depara
  when agents must catch up. Repo wins only when owner orders `sync-kit.mjs`.

## 0.16.0 — 2026-07-21

- **Kit↔Supabase depara** — `partials/kit-depara.md` + spell `/kit-depara`.
  `truth-keeper` responds to `depara kit supabase` with live diff, classifies
  drift from git, `/kit` Save, and **DataGrip** edits; asks owner before sync.
- **`check-kit-drift.mjs --depara`** — Postgres via `DATABASE_*` (DataGrip path),
  provenance hints (file mtime vs `updated_at`), human-readable report.

## 0.15.0 — 2026-07-21

- **Kit navigation** — tool-agnostic bootstrap contract
  (`partials/kit-navigation.md`): cold-start read order, skill/agent/partial
  parsing, adapter rules (Cursor / Claude). No per-IDE skill SoT.
- **Skill `kit-entry`** — cold start without chat history; links navigation
  partial (does not duplicate it).
- **`scripts/sync-cursor-adapters.mjs`** — regenerate thin `.cursor/skills/`
  from `gestalt-kit/skills/`.
- Wire: `maestro`, `truth-keeper`, `declutter`, `sot-matrix`, `architecture`,
  `AGENTS.md`, `repo-consistency`, `gestalt-vault/README.md` stub.

## 0.14.0 — 2026-07-20

- **UC esteira** — owner confirmation gate before coding
  (`partials/uc-esteira.md`). SoT for UC intent/runtime = live
  `portfolio.use_cases` (+ steps, requirements); vault = authoring replica
  (DB wins on drift). Durable mark via `metadata.esteira` (no new column).
- Wire: `maestro`, `truth-keeper`, `persona-crafter`, `declutter`,
  `implement-deviante-uc`, `write-use-case`, `use-cases-surface`, `gamifier`,
  `/ship-quest`, `sot-matrix`, architecture.
- Spell `/uc-gate` — fire the checklist for one ABP id.

## 0.13.0 — 2026-07-20

**Breaking path move** — open Obsidian at `gestalt-kit/vault/` (not root vault).

- Unify hub knowledge under `gestalt-kit/`: `vault/` (ex-`gestalt-vault`),
  `plans/` (ex-`doc/sprint-plan*`, `UC-Template`), Obsidian config under vault.
- Stubs: `doc/README.md`, `gestalt-vault/README.md` → redirects only.
- `scripts/sync-vault.mjs` → `gestalt-kit/vault`; kit sync still skips vault/plans.
- Folder plant SoT in `docs/architecture.md` (architect owns); declutter /
  truth-keeper verify live dirs against the plant.
- SoT matrix: kit + vault are one home; hunt replicas outside the kit.

## 0.12.3 — 2026-07-20

- Add `interaction-designer` (motion / expand-collapse / flicker audits).
- `ui-designer`: spacing, button variants, informative delete, product-themed
  quest log; hand motion to interaction-designer.

## 0.12.2 — 2026-07-20

- Architect: endorse **Kit↔DB reconciliation** (poll-based Observer) — not
  GoF push Observer. `scripts/check-kit-drift.mjs` + MCP `md5(body_md)`.
- Update surfaces for kit text: **only** `gestalt-kit/` + `portfolio.kit_docs`.
- `declutter`: always scan Cursor / Claude worktrees / Obsidian / `doc/agents`
  for full skill copies (thin adapters OK).

## 0.12.1 — 2026-07-20

- `declutter`: always scan tmp*, kit↔vault bleed, stubs; prefer Supabase MCP
  before deleting data recipes. Clarify kit vs vault must not merge.
- SoT matrix: `/kit` owner edits persist in `portfolio.kit_docs`.

## 0.12.0 — 2026-07-20

- Portfolio `/kit` CMS: `portfolio.kit_docs` + `scripts/sync-kit.mjs`;
  nav link; owner edit like Cases. SoT: authoring = `gestalt-kit/`,
  runtime report = live `kit_docs` (`truth-keeper` / `sot-matrix`).

## 0.11.0 — 2026-07-20

- Partial `portfolio-typography.md`: Carbonot = display titles **or** one
  chrome size; GT Planar for readable copy. Linked from `ui-designer` + SoT matrix.
- Portfolio UC Save: sticky toolbar (no mid-viewport float); contrast via
  color, not opacity-only; focus ring + larger hit target.

## 0.10.0 — 2026-07-20

- New agents: `maestro` (conducts roster, strategy questions, kit structure
  audit for skills/partials/spells) and `content-strategist` (publish
  strategy grounded in vault + OOUX; owner fill-in for truncated mandate).
## 0.9.0 — 2026-07-20

- New agent: `data-guardian` — verifies live Supabase before seed/SQL dump
  deletion; if DB MCP is unauthenticated, asks the owner to connect; audits
  commits/deploys so secrets are not exposed.
## 0.8.0 — 2026-07-20

- **Canonical home is gestalt-kit** — skills (`SKILL.md` + `reference.md`),
  agents, partials, and `docs/architecture.md` live here only.
- `doc/agents/` reduced to a redirect README; removed duplicate bridges
  (`doc/flashbrix`, `doc/milebrick`, `doc/harpia`, out-of-scope product stubs).
- `declutter` agent finished — owns duplicate-doc cleanup and enforces the
  single-home rule.
- `.cursor/skills/` adapters retargeted to `gestalt-kit/skills/…`.

## 0.7.0 — 2026-07-20

- Day-1 kit focus: active scope **IO + DV only**; Flashbrix forbidden;
  Milebrick/Harpia out of scope (`partials/active-scope.md`,
  `repo-consistency` skill).
- `architect` replaces `system-designer` — owns SoT
  `doc/agents/architecture.md` (wired in `truth-keeper`).
- New agent: `ui-designer` — product styling / contrast over raw ADS.
- New skills: `prefer-existing-files`, `repo-consistency` (kit + Cursor
  adapters).
- Sprint plan day 1 = agents + cleanup; JDBC work shifts to day 2.

## 0.6.0 — 2026-07-20

- New agents:
  - `persona-crafter` — product personas that complement ABP user stories;
    aligns with ORCA Roles when a Hub exists.
  - `polyrepo-shipper` — routes commit/push across registered remotes
    (hub, portfolio, deviante-api/web); only dirty repos; asks when ambiguous.

## 0.5.0 — 2026-07-20

- New agents:
  - `truth-keeper` — unique source-of-truth map per domain; observer-style
    drift checks (vault / schema / Supabase / site / screenshots / sprint
    nicknames vs `portfolio.quests`).
  - `system-designer` — always check for a recommended in-repo or stack
    pattern; keep layered architecture coherent.
  - `ooux` — ORCA facilitator aligned to Notion ORCA hubs + OOUX masterclass
    practices; owner is stakeholder/decision-maker.

## 0.4.0 — 2026-07-20

- New skill: `gamifier` (`skills/gamifier/SKILL.md`) — the public, always-on
  quest log widget (`ui/gamifier/`) shown on alander.io and
  deviante.alander.io, plus the Postgres trigger that turns UCs into quests
  and auto-syncs their status. Mirrors `doc/agents/gamifier.md`.
- `ship-quest` updated: Deviante quests moved onto `portfolio.quests`
  (Supabase) alongside IO/MB/HA, and quests ending in `-spec` are now
  excluded from the manual-update step — the database keeps those in sync
  on its own.

## 0.3.0 — 2026-07-20

- `ship-quest` now points at `portfolio.quests` (Supabase) as the live
  source of truth, with `content/gestalt-roadmap.json` demoted to
  fallback/bootstrap only — matches the new `RoadmapProvider` wiring in
  Portfolio.

## 0.2.0 — 2026-07-20

- New agent scaffolds for the Deviante 10-day sprint: `deviante-frontend`,
  `deviante-backend`, `database-integrations`. Deliberately incomplete —
  frontmatter is filled in, the body's scope/conventions/boundaries
  sections are left for the owner to write.
- New spell: `ship-quest` (`skills/ship-quest/SKILL.md`,
  `disable-model-invocation: true`) — turns the "close out a quest"
  checklist from the 1-month plan into a `/ship-quest` command.
- Introduced "spell" as this kit's name for a skill that's command-only
  (never auto-invoked by Claude) — documented in the README.

## 0.1.0 — 2026-07-20

- Initial version: 11 skills mirroring the `.cursor/skills/` adapters (kept
  in sync — both point at `doc/agents/`, nothing duplicated).
- New `uc-scaffolder` agent: drafts a first-pass ABP use case from a short
  description, preloading `write-use-case` + `gestalt-context`.
- Fixed a broken relative-path bug in 8 of the 9 original `.cursor/skills/`
  files (links were one directory short and silently pointed nowhere).
- Extracted the "no clutter" owner-preference text, previously duplicated in
  `doc/agents/gestalt-context.md` and `.cursor/skills/gestalt-context/SKILL.md`,
  into `doc/agents/partials/owner-preferences.md`.

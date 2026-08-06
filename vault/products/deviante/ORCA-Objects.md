# Deviante ORCA — Objects (v1.0 committed scope)

[[Deviante]]

> **Vocabulary source of truth:** Deviante **ORCA Hub 2.2** in Notion
> (page `27b5fc72-4940-8229-a5c6-01c46e28946f`, Objects data source
> `fe85fc72-4940-838a-bbc5-879703bc5e8f`). This file is a **repo-side mirror**.
> The Hub wins for naming. See [[ooux-vocabulary]]. The agnostic, agent-facing
> version of this model lives in `deviante/README.md`.

> **VOCABULARY FLIP DECIDED 2026-07-28 (rename pending).** Owner confirmed the
> product/UC vocabulary: the **normalized concept = Operation** (curated, global,
> created via upload); **raw log labels = alias** (an attribute of Operation);
> **activity = only the concise description field**, not an object. This *inverts*
> the built DB names. **DB/code rename is deferred — do not run it now**; the UCs
> lead the vocabulary. Until the rename lands, read the "built model" table below
> with: **Operation ↔ DB table `activities`**, **alias ↔ DB table `operations`**.
> v1 is **main-flows-only** (PIBITI/thesis) — no alternative/error flows yet.
> (Supersedes the earlier "keep built vocabulary" note from the same day.)

## Built model (what code + UCs already use)

| Object | Table (`deviante.*`) | Rows | Role | UCs |
|--------|----------------------|------|------|-----|
| **Process** | `processes` | 5 | Manufacturing workflow (company-owned) | UC2 |
| **Activity** | `activities` (+ `process_activities` junction) | 25 (+25) | **Normalized logical concept** several raw operations converge into; global across a company's Processes (many-to-many); manager-maintained normalization dictionary | UC3 |
| **Operation** | `operations` | 32 | **Raw log label** from an outsourced event log (the alias variants); each maps to one Activity | UC5, UC6 |
| **Trace** | `traces` | 7,564 | A process instance / execution | UC7–UC11 |
| **Trace event** | `trace_events` | 30,106 | **Occurrence** — one event in a trace (timestamp, duration). Backing data, not a hero screen | — |
| **Event Log** | `event_logs` | 4 | One upload (CSV/XES) into a Process | UC4 |
| **Analysis** | `analyses` | seeded | An IPDD/ADWIN drift run + filter provenance + outputs. Listed on the dashboard; reopen loads saved `result_json` | UC12, UC13 |
| **Maintenance** | *(no table yet)* | — | Recommended / scheduled action | UC11, UC14/UC15 (out of window) |
| **Equipment** | `equipment` | 0 | Machine / asset. Deferred this window | (UC14/UC15) |
| **Manager / User** | `managers` / `users` | 3 / 3 | Actor | UC1 |

**Future (next version):** **Monitoring** — analysis over machine parameters
(sensor histories / logs) across a time period → **RUL** or **temperature-drift**
IPDD graph. Ties to Equipment. Not in the 5-day window.

## Hub-committed five objects

The ORCA Hub Objects DB currently lists five: **PROCESS, OPERATION, EQUIPMENT,
MAINTENANCE, ANALYSIS**. Note **Activity is not a row in the Hub Objects DB**,
yet it is a first-class object in code + UCs (UC3, `activities`). Sync follow-up:
reconcile the Hub Objects rows with the built model above (Activity, Trace, Event
Log, Trace event exist in code/UCs but not all as Hub Object rows). Do not rename
code to match the Hub without an owner decision.

## Key object detail

### Activity (normalization concept, manager-maintained)
- **What:** the clean logical activity several raw Operations converge into
  (UC3). Concise human name (~≤8 words). Global across a company's Processes via
  `process_activities` (many-to-many); can be orphan (in 0 Processes).
- **Aliases:** realized as the raw **Operations** mapped to it — different
  outsourced logs use different labels for the same physical activity (UC5/UC6).
- **Not a hero UX object** (owner 28/07): a support/normalization surface, not a
  big browsable destination.

### Operation → Activity mapping
- At upload, raw operation labels are matched to Activities; unmatched fall to
  manual mapping in the upload modal (UC5/UC6). This is the alias mechanism.

### Analysis (v1.0 — the core new work)
- **Is:** the IPDD/ADWIN engine (`FastAPI /detect`) producing drift outputs
  (`processed_values`, `outlier_indices`, `smoothing_window`, `drifts[]`).
- **Scope / provenance (owner 28/07):** the Process screen lets the Manager
  filter before running — choose cycles, remove traces, pick activities, or a
  **single Activity**. After the run, the Analysis screen must show **exactly
  what it was run on** (the filters/scope), persisted with the result.
- **Single-activity:** natively supported — the ADWIN flow already filters one
  activity and builds its per-trace sojourn-time series.
- **Gap:** no `analyses` table yet; filter→`values[]` wiring and the provenance
  screen are unbuilt. Gap work proceeds only from validated UCs.

## Decisions log

- **2026-07-28 (later) — Vocabulary FLIPPED to product language; name map
  confirmed by owner.** Normalized concept = **Operation** (was `activities`),
  raw labels = **alias** (was `operations`), activity = concise description field
  only. Main creation of Operations = the log upload (UC4). UC3 rewritten to
  "Maintain Operation" (curation; upload is primary creation); UC5 realigned to
  "map raw labels/aliases → Operations"; UC6 retitled "Match Alias with
  Operation". **DB/code table rename deferred** (owner: don't alter now, focus on
  UCs). v1 = main-flows-only (PIBITI/thesis). This supersedes the earlier same-day
  "keep built vocabulary" decision below.
- **2026-07-28 — Vocabulary ratified to the built system.** Keep Process /
  Activity / Operation (raw label) / Trace / Trace event as in code + UCs. No
  mid-sprint rename. Supersedes the earlier 28/07 "OPERATION = occurrence" and
  "ACTIVITY = support entity with aliases" drafts (they inverted the built names).
- **2026-07-28 — Activity is not a hero UX object.** It stays the manager-
  maintained normalization concept (UC3); its aliases are the raw Operations
  mapped to it.
- **2026-07-28 — Activity ↔ Process is many-to-many, company-scoped**, already
  implemented via `process_activities`. Enables global analysis of one activity
  across every Process it appears in.
- **2026-07-28 — Analysis = IPDD/ADWIN output object with filter provenance**,
  can target a single Activity. Needs an `analyses` table.
- **2026-07-28 — P-F curve is part of the Analysis output.** Maintenance domain
  concept: the interval from **P** (potential-failure point = anomaly onset =
  ADWIN drift start / `anomaly_start_index`) to **F** (functional failure).
  Maintenance must fall inside the P-F interval, so the Analysis screen must make
  that interval and the recommended maintenance window **evident** to the Manager.
- **2026-07-28 — UC12 validated + written to Supabase.** Owner approved the
  Analysis-provenance + single-activity + P-F changes; UC12 updated in
  `portfolio.use_cases` (status `ready`, `metadata.priority=p0`). UC status
  vocabulary is the enforced DB set `draft/ready/shipped/deprecated`; Supabase is
  canonical (vault md sync retired for Deviante).
- **2026-07-28 — UC13 validated + written to Supabase.** Params trimmed to the
  real engine knobs: sensitivity (ADWIN `delta`) + baseline scope (all vs
  reference-only, ties UC8) + minimum observations; algorithm fixed, smoothing
  window auto/read-only. Reached via a CTA opening a dedicated adjustments
  section — overlay on desktop, full section on mobile. Params + scope snapshotted
  per run. `status=ready`, `priority=p0`.
- **2026-07-28 — Monitoring / RUL / temperature-drift = future version**, tied to
  Equipment; out of the 5-day window.
- **2026-07-28 — Seed all objects for 1.0** from Luiz's research datasets
  (`deviante/Adaptive-Detection-of-Performance-Related-Temporal-Drifts-main/`)
  and `deviante/docs/relatorio/codigos/dataset_manufacturing/` + `real_dataset/`.
  Preserve IPDD provenance (Denise, Edson, Luiz) — hard gate #6.

## Open items

- **Employee / resource:** owner mentioned employees performing operations —
  attribute (resource on trace_event) or Role? Not a new object.
- **Hub sync:** reconcile the Hub Objects DB with the built object set.
- **UC status vocabulary:** single `status` text today (all "ready"); owner wants
  richer owner-controlled states (see `deviante/README.md`).

## Do not

- Do not rename code/DB to match the morning draft — the built vocabulary is SoT.
- Do not start Analysis gap work before the relevant UCs are validated by the owner.
- Do not treat this mirror as vocabulary authority over the Notion Hub.

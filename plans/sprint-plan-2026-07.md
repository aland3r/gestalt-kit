# Deviante - 10-day delivery plan

**Window:** 23/07/2026 to 01/08/2026
**Target:** Deviante UC2-UC13 happy path running on the real deployed product.
**Acceptance:** the owner completes the production journey and says
**"amei, next"** for each slice.

**v1 is MAIN-FLOWS-ONLY (PIBITI / thesis).** v1.0 is dedicated to the
CNPQ-PIBITI evidence for the owner's thesis, so every UC implements ONLY its
main (happy-path) flow for now — no alternative or error flows. UC descriptions
and specs carry the primary flow only until v1 ships.

This file is the current roadmap. It replaces the stale 21/07 schedule. Git
history preserves the prior plan.

**Stewardship:** Alander builds, publishes, and decides alone. Eduardo Loures
and Luiz Picolo are research partners and qualified testers. Agents are
assistive roles, not human delivery owners.

## Scope

### In this 10-day window

- UC2-UC6: shared Process, global Activity catalog, real CSV/XES upload,
  explicit mapping, and generated process graph.
- UC12-UC13: IPDD/ADWIN integration and the minimum confirmed analysis
  parameters.
- UC7-UC11: investigate detected drift and persist the Manager's decisions.
- Real Supabase data, Kotlin persistence, FastAPI computation, React UI, and
  production deployment.
- PT-BR UI copy and incremental visual consistency with the latest Figma Make
  reference.

### Outside this window

- UC14-UC15.
- Alternative and error flows.
- Equipment/machine and 3D visualization.
- Portfolio work.
- Broad rebranding or a full redesign.
- Predictive-maintenance/RUL features that are not needed by UC2-UC13.

## Product decisions

- Every authenticated Deviante user can list, open, and edit every Process.
- Only an Owner can delete a Process. Deletion requires the exact Process name
  and an explicit typed deletion intent.
- Activities form one global catalog shared by all Processes.
- Event Log upload accepts CSV and XES.
- Event Log upload stays on the Process screen. One modal contains two
  modules: upload/parse and, after parsing, mapping extracted operation labels
  to global Activities. It is not a wizard or a separate page.
- Mapping is explicit. The system does not silently create Activities.
- The Process graph becomes available only after required mappings are
  validated and the modal closes. A newer incomplete upload does not replace
  the last fully mapped graph.
- Alander uses the `owner` runtime role. Eduardo Loures and Luiz Picolo use
  `mentor`; they can test shared data but cannot delete Processes.
- Equipment association is deferred.
- FastAPI is stateless computation for process mining and IPDD/ADWIN. Kotlin
  owns business rules and persistence.
- UI is PT-BR; code and knowledge documentation are English.
- Functionality precedes visual refinement, but every touched screen should
  stay consistent with the latest Figma Make version.

## Canonical fixtures

| Purpose | Fixture | Known behavior |
|---|---|---|
| Real CSV upload and mapping | `real_dataset/Prod1Torno.csv` | 13,053 events, 3,282 traces, 12 labels |
| Stable XES control | `dataset_manufacturing/ST_01.xes` | 500 traces, 4 labels, no intended drift |
| Simple XES drift | `dataset_manufacturing/DR_01.xes` | 500 traces, one intended drift |
| Strong visual drift demo | `dataset_manufacturing/DR_MS_20.xes` | Five intended drifts; existing matrix PNG/CSV |

`adwin_dataset.py` is an offline ADWIN evaluator, not an upload dataset. It
must be wrapped or adapted without rewriting the research algorithm.

## Delivery rhythm

For each slice:

1. Load the live UC, steps, and ACs from Supabase.
2. Ask only questions that block the current happy path.
3. Persist owner corrections and read them back.
4. Obtain explicit permission to implement.
5. Implement a vertical Kotlin/FastAPI/React slice.
6. Run a local smoke test, deploy, and repeat the journey in production.
7. Present it to the owner.
8. After "amei, next", mark the UC accepted and advance.

## Ten days

| Day | Functional delivery | Production acceptance |
|---|---|---|
| **1 - UC2** | Reconcile deploys and access. Authenticated testers see/edit shared Processes. Only Owner deletes, with strong typed confirmation and cascading cleanup. | Alander publishes the slice; Eduardo and Luiz can test the same Process, while only Alander's Owner account can delete a disposable one. |
| **2 - UC3** | Global Activity catalog with authenticated Kotlin CRUD, database persistence, and an explicit Activity set for each manually defined Process. | Create/edit an Activity, reuse the same catalog from two Processes, and add/remove it from each defined model without inventing log metrics. |
| **3 - UC4 upload** | Process-screen modal with upload and mapping modules. Upload real CSV/XES, FastAPI parses, Kotlin persists the Event Log, occurrences, timestamps, durations, and traces. | `Prod1Torno.csv` and `ST_01.xes` both reveal mapping in the same modal with expected counts. |
| **4 - UC5/UC6 mapping** | Operation label on the left, global Activity card on the right, per-row edit/validation controls, and safe complete-batch persistence. | Map every label, validate, reload, and retain the associations. |
| **5 - UC2-UC6 journey** | Gate the graph on a fully mapped log, retain the last valid graph during a new upload, support mapping resumption, and fix blocking UI inconsistencies. | Alander publishes authentication -> Process -> upload -> mapping -> graph; Eduardo and Luiz can run the journey as research testers. |
| **6 - IPDD integration** | Preserve research provenance, define the ADWIN input/output contract, persist analysis results, and only now review UC12/UC13. | Reproduce a known stable/drift result using the canonical fixtures. |
| **7 - UC12/UC13** | Confirm essential filters/parameters and wire the primary drift-analysis CTA through Kotlin to FastAPI. | `ST_01` acts as control and `DR_01` identifies the intended drift. |
| **8 - UC7/UC8** | Select a drift point, inspect the affected trace and events, and persist a reference execution. | Open a real flagged trace and retain its reference decision after reload. |
| **9 - UC9/UC10/UC11** | Persist dismissal, baseline exclusion, and maintenance recommendation happy paths. | Each decision survives reload and appears in the investigation context. |
| **10 - release pass** | End-to-end regression, production fixes, PT-BR copy repair, and focused visual alignment on touched surfaces. | Owner completes UC2-UC13 in production and closes the cycle with "amei, next". |

## Hard technical gates

1. **Temporal event data:** FastAPI already calculates duration/end data, but
   current ingestion discards part of it. Persist actual start, end, and
   duration before treating the graph or future ADWIN input as scientifically
   valid.
2. **Authorization:** current code lists shared Processes inconsistently and
   still scopes open/edit/delete by creator. Fix authorization as one UC2
   change, not route by route.
3. **Mapping integrity:** authenticate every Activity/mapping route, verify
   every Operation belongs to the Process, and reject incomplete or
   cross-Process mapping batches.
4. **Defined/observed separation:** `process_activities` records the Activity
   set selected by the user for the manually defined model. `/graph` remains
   the observed process derived only from mapped log events. An Activity that
   has no occurrence in the log must never receive inferred frequency,
   duration, or analysis eligibility.
5. **Graph gate:** show the latest fully mapped log. A new incomplete upload
   must not replace a previously valid graph.
6. **Research provenance:** preserve attribution to the IPDD research by
   Denise, Edson, and Luiz. Wrap the existing implementation; do not rewrite
   the algorithm as product code without an explicit research decision.
6. **Deployment truth:** identify and document the actual web, Kotlin, and
   FastAPI deployment pipelines before Day 1 acceptance. Do not rely on
   assumptions about Fly.io or Vercel.

## Scope fallback

If the schedule slips, reduce visual polish and UC9-UC11 depth first. Do not
sacrifice real upload, temporal data, mapping integrity, graph generation, or
the ADWIN integration.

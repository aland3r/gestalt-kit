# Debug drift detection workflow

## Non-negotiable invariants

- Start from a Process, but allow the numeric series to target the whole
  process, one or more Activities, and a filtered Trace population.
- Express the projection positively. Never infer `process` versus `activity`
  only from excluded IDs. A single Activity must remain representable when it
  is the only mapped Activity.
- Keep projection, population, event-log version, and ADWIN parameters
  orthogonal and persist the exact applied configuration with the result.
- Use stable UUIDs across recomputation. Use series indexes only to locate
  points inside one detector response.
- Validate the minimum observation count after all filters. Do not silently
  substitute the process-wide count.
- A UI filter is pending until a new run succeeds. Never relabel an old result
  by merely updating saved filter IDs.
- Do not settle `cycle/window` as a domain entity while research validation is
  pending. Activity and Trace filtering are already supported requirements.

## 1. Capture the case

Record expected behavior, actual behavior, visible error, Process ID,
Analysis ID, selected Activity/Trace IDs, event-log ID/file, and whether the
result is fresh or reopened. Preserve existing worktree changes.

Load:

- [Deviante domain](../deviante-domain/reference.md), especially analysis scope
- `gestalt-kit/docs/architecture.md` process-mining/drift boundary
- UC12/UC13 live SoT when available; use the vault only as a replica

Treat a semantic/acceptance-criteria change as UC work: stop and route it
through `/uc-gate`. A regression that restores confirmed behavior may proceed
as a bug fix.

## 2. Assemble only the needed team

Run independent backend, frontend, and truth checks in parallel when the host
supports subagents, then synthesize once.

| Role | Enter when | Responsibility |
|---|---|---|
| `maestro` | Always | Conduct, constrain scope, synthesize |
| `truth-keeper` | Always | Confirm UC12/UC13 and resolve SoT drift |
| `deviante-backend` | Always | Ktor contract, series builder, FastAPI boundary |
| `deviante-frontend` | UI or replay involved | Graph identity, request, draft/applied state |
| `architect` | Boundary change proposed | Approve the smallest coherent contract |
| `database-integrations` | Data/schema evidence points there | Mapping, duration, provenance storage |

Do not create a new persona for this workflow. The command coordinates the
existing specialists.

## 3. Reproduce a differential matrix

Use the same parsed event log and detector parameters for every row:

| Run | Projection | Population |
|---|---|---|
| A | Whole-process/whole-trace duration | All valid Traces |
| B | Two mapped Activities | Same Traces |
| C | Exactly one Activity | Same Traces |
| D | Same Activity as C | One or more Traces filtered |

When the Process has only one mapped Activity, run both A and C. Their request
configuration must differ even if their observation counts happen to match.

## 4. Build the boundary ledger

Follow one run end to end and record the first divergence:

```text
graph intent
-> visual node identity/activityId
-> navigation or draft Analysis
-> HTTP payload
-> RunAnalysisRequest
-> Process + eventLog + scope membership validation
-> AnalysisRepository series
-> values sent by MiningClient
-> FastAPI /detect response
-> Kotlin index/provenance projection
-> analyses persistence
-> reopen/rendered applied configuration
```

At each boundary record configuration, observation count, and a redacted
sample or deterministic hash of the numeric series. Do not log secrets,
tokens, proprietary full datasets, or personal data.

Check specifically:

- the selected graph node retains `activityId` and the handler consumes it;
- Activities and Traces belong to the same Process and chosen event log;
- the UI does not combine Activities from all uploads with a latest-log-only
  backend run;
- mapped Operations and `trace_events.duration_seconds` exist;
- a one-Activity projection aggregates all aliases mapped to that Activity,
  once per Trace and in chronological order;
- the post-filter observation count satisfies the confirmed scientific/product
  threshold;
- detector indexes are converted consistently to stable Trace IDs;
- saved `scope`/`appliedConfig` describes the series that actually ran.

## 5. Classify the first divergence

Classify it as one of: UI intent, transport/DTO, membership validation,
series construction, mapping/duration data, detector, result projection,
persistence/reopen, or presentation. Fix the earliest faulty boundary; later
symptoms usually disappear once its contract is restored.

Prefer an explicit contract shaped like:

```json
{
  "eventLogId": "uuid",
  "projection": {
    "kind": "process_duration | activity_sojourn",
    "activityIds": ["uuid"]
  },
  "population": {
    "excludedTraceIds": ["uuid"]
  },
  "algorithm": {
    "method": "adwin",
    "treatment": "treated",
    "delta": 0.002
  }
}
```

Exact DTO names may follow repository conventions. Preserve the separation of
axes and return the resolved event log, Activity identities/labels, Trace
selection, observation count, and algorithm parameters as applied provenance.

## 6. Fix and prove

Only with `--fix`:

1. Patch the earliest divergent boundary with the smallest coherent change.
2. Add a regression test at that boundary and one integration assertion across
   the adjacent boundary.
3. Cover: process scope; exactly one Activity including a sole Activity; aliases
   mapping to one Activity; filtered Traces; foreign/stale IDs; insufficient
   post-filter observations; and save/reopen provenance.
4. Replay matrix A-D with the same log and parameters.
5. Report the before/after ledger and ask the owner to replay the original real
   case when local fixtures cannot reproduce proprietary data.

## Output contract

Return:

```text
Symptom:
Expected scope:
First divergent boundary:
Evidence:
Root cause:
Series before/after (count + redacted sample/hash):
Fix or proposed fix:
Tests:
UC/SoT impact:
Remaining real-data replay:
```

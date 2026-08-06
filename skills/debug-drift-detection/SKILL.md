---
name: debug-drift-detection
description: >-
  Owner-invoked command that diagnoses and, with --fix, repairs Deviante
  IPDD/ADWIN drift-analysis failures across graph selection, API scope,
  event-log data, time-series construction, detection, persistence, and UI.
  Use explicitly as /debug-drift-detection when process-, Activity-, or
  Trace-filtered analysis returns an error, ignores the selected scope,
  produces suspicious results, or reopens with inconsistent provenance.
disable-model-invocation: true
---

# Debug drift detection

Run the evidence-first workflow in [reference.md](reference.md).

Invocation:

```text
/debug-drift-detection <symptom> [--process <uuid>] [--analysis <uuid>]
  [--activity <uuid|label>] [--trace <uuid>] [--fix]
```

Without `--fix`, remain read-only and return a diagnosis plus the smallest
safe correction. With `--fix`, patch only after locating the first divergent
boundary, then add a regression test there and replay the scope matrix.

Preserve this invariant throughout: a Process is the root context, not
necessarily the statistical unit. The projection (process duration or
Activity sojourn) and the Trace population are separate, explicit inputs.

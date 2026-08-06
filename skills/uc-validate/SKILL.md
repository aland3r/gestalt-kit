---
name: uc-validate
description: >
  Cheap multi-agent checklist for one UC on the esteira — coordinates
  product-manager, architect, and gamifier without spending their model
  budget on coordination. Complements /uc-gate (the explicit-yes gate);
  does not replace it. Use before or after implementing a UC.
disable-model-invocation: true
argument-hint: [abp-id] [--gates product-manager,architect,gamifier]
---

<!--
Command: only the owner invokes (/uc-validate). Full mechanics:
gestalt-kit/skills/uc-validate/reference.md
Gate contract this complements: gestalt-kit/partials/uc-esteira.md
-->

Run the **modular validation checklist** for `$0` (e.g. `ABP-DV-UC6`).
Default gates (if `--gates` omitted): `product-manager,architect` before
code, `gamifier` after accept.

1. Read [reference.md](reference.md) for the orchestration mechanics
   (background-agent launch, per-gate model routing).
2. Load the UC live via **Supabase MCP** — never from a cached/plan-file
   claim (see [uc-esteira.md § Source of truth](../../partials/uc-esteira.md#source-of-truth)).
3. For each requested gate, launch the matching agent as a **background
   Agent** call (`product-manager`, `architect`, or `gamifier` — each
   keeps its own configured model; this command's own coordination stays
   on the cheapest available model, not theirs).
4. Aggregate results into one checklist, most-blocking first.
5. Report to the owner. This command **never** sets
   `metadata.esteira.review_status` itself — that write only happens via
   `/uc-gate` after the owner's explicit "confirmado".

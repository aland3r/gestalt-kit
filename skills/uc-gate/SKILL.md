---
name: uc-gate
description: >
  Owner confirmation gate for one use case on the esteira. Loads the live DB
  spec card, asks for yes/edits, marks metadata.esteira. Use when starting or
  confirming a UC before coding.
disable-model-invocation: true
argument-hint: [abp-id]
---

<!--
Command: only the owner invokes (/uc-gate). Full contract:
gestalt-kit/partials/uc-esteira.md
-->

Run the **UC esteira gate** for `$0` (e.g. `ABP-DV-UC4`). If `$0` is empty,
propose the next `unchecked` UC for IO/DV from live DB.

1. Read [gestalt-kit/partials/uc-esteira.md](../../partials/uc-esteira.md).
2. Load the UC from **Supabase MCP** (`portfolio.use_cases` + steps +
   requirements + linked quests). If MCP is unavailable: stop; ask me to
   connect MCP or paste from `/cases` — do not invent the spec.
3. Show the **spec card** (Why/What/Bounds, actor, flows, ACs, personas,
   quests that will move, vault↔DB drift).
4. Wait for my **confirm** or **edit** instructions. Prefer edits on
   `/cases` or DB. Default on drift: **DB wins**.
5. After explicit yes, set `metadata.esteira.review_status = spec_confirmed`
   (and `spec_confirmed_at`) via MCP. If you cannot write DB, ask me to.
6. Tell me the UC is cleared for `implement-deviante-uc` (or IO path) —
   do **not** start coding inside this command unless I also ask to implement.

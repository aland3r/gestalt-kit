---
name: gamifier-sync
description: >-
  Audit quest log against Supabase, truth-keeper, and sprint plan. Detect stale/ghost quests and propose next step coherent with product-manager + roadmap.
disable-model-invocation: true
argument-hint: "[quest-id] [--fix]"
---

<!--
Command: owner invokes only (/gamifier-sync). Detects quest log drift.
Reads: portfolio.quests (Supabase), portfolio.use_cases (spec), sprint-plan-2026-07.md (roadmap),
truth-keeper findings. Proposes coherent next step.
-->

Audit quest `$0` (or all if empty) for consistency with live Supabase + roadmap:

1. Load live **quest row** from `portfolio.quests` (quest_id, status, product_code, linked_uc).
2. Load **UC spec** from `portfolio.use_cases` (if linked) — check `metadata.esteira.review_status`.
3. Cross-check **sprint plan roadmap** (`gestalt-kit/plans/sprint-plan-2026-07.md`) — day X, phase Y, stage.
4. Consult **truth-keeper findings** — any drift between repo SoT and Supabase?
5. Report **status matrix** (quest | UC | spec status | roadmap day | drift?).
6. **Propose next step** — if drift detected, ask product-manager + architect which source wins.
7. If `--fix` flag, apply corrections (ask for approval before touching DB).

**Purpose:** Catch ghost quests (orphaned), stale quests (linked UC changed), and sync quest log with owner's actual roadmap intent.

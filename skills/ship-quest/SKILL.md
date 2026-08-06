---
name: ship-quest
description: Marks a quest done and syncs the roadmap docs after you've verified it shipped.
disable-model-invocation: true
argument-hint: [quest-id] [product]
---

<!--
This is a "command": our own word for a skill that only YOU invoke by name
(never Claude, automatically) — set with `disable-model-invocation: true`.
It's just a skill with that one flag; "command" is vocabulary, not a new
mechanism (renamed from "spell" 21/07/2026 — same thing, plainer name: this
is literally a real slash command). Good candidates for commands: anything
with a side effect whose timing you want to control (see
`gestalt-kit/skills/product-progress/reference.md` — this one exists because
"close out the quest" was already a repeated manual checklist in the
1-month plan).
-->

Close out quest `$0` for product `$1`:

1. Confirm with me that the quest is actually shippable and verified — do
   not mark anything done on trust alone. If this quest belongs to a UC on
   the esteira, the UC should already be owner-confirmed
   (`gestalt-kit/partials/uc-esteira.md`) and ACs checked.
2. **Portfolio and Deviante quests (IO/DV/MB/HA) live in Supabase now**
   (`portfolio.quests`, since 20/07/2026 — see `gestalt-kit/skills/product-progress/reference.md`).
   Update the row with a SQL `UPDATE portfolio.quests SET status = 'done',
   updated_at = now() WHERE product_code = '$1' AND quest_id = '$0';` —
   editing `content/gestalt-roadmap.json` / `deviante/web/src/lib/roadmap.js`
   alone no longer changes the live site, those files are fallback/bootstrap
   only now. **Exception:** don't hand-edit a quest whose `quest_id` ends in
   `-spec` — those are `auto_synced = true` and the database already flips
   them to `done` when you set the linked UC's `status` to `ready`/`shipped`
   in `/cases` (see `gestalt-kit/skills/gamifier/reference.md`); a manual UPDATE would just be
   overwritten (or redundant) the next time the UC is saved.
3. Set the next quest in the same phase to `active` the same way — there
   should be exactly one `active` quest per phase at a time (see
   `gestalt-kit/skills/product-progress/reference.md`). Prefer next quests
   that are still **in scope** for the confirmed UC (gamifier post-accept).
4. If you also want `content/gestalt-roadmap.json` to stay readable as a
   point-in-time snapshot (not required for the live site to work), update
   it too — but say so explicitly, it's optional now.
5. Update the matching row in the product's `docs/roadmap.md` table.
6. If MCP was unavailable for the SQL steps, say so and ask — do not claim
   the DB changed.
7. Tell me exactly what changed (SQL run + any files edited), in one short
   list — no extra narration.

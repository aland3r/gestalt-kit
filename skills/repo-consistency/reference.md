# Repo consistency (no junk)

Keep the workspace **consistent and tied together**. Agents and skills must
enforce these rules — not leave ghost products in Obsidian or docs.

## Rules

1. **Active scope only** — follow
   [partials/active-scope.md](../../partials/active-scope.md):
   Portfolio + Deviante. Milebrick and Harpia are out of scope.
2. **No Flashbrix** — forbidden name. Never treat it as a second product
   beside Milebrick. Strip mentions on sight.
3. **One name per thing** — product codes, ORCA objects, quest IDs, and
   folder names must agree. If Obsidian shows two products for one idea,
   that is junk: fix links/paths, do not add a third doc.
4. **No orphan docs** — new content prefers existing files
   ([prefer-existing-files.md](../prefer-existing-files/reference.md)).
5. **No clutter UI/code** —
   [partials/owner-preferences.md](../../partials/owner-preferences.md).
6. **SoT map** — when two artifacts disagree, use `truth-keeper` and the
   domain SoT; do not keep both "for history" in active trees.
7. **Kit navigation** — one knowledge home; thin IDE adapters only —
   [partials/kit-navigation.md](../../partials/kit-navigation.md).
   Regenerate Cursor adapters: `node scripts/sync-cursor-adapters.mjs`.

## Checklist (before finishing a docs/agents task)

- [ ] Grep for `flashbrix` / `Flashbrix` / `ABP-FL` — zero hits in content
      you touched (and fix strays you find nearby)
- [ ] No new Milebrick/Harpia "active" roadmap or sprint work
- [ ] No duplicate product MOC under a second name
- [ ] Links point at `gestalt-kit/vault/` / kit SoT, not stale `doc/flashbrix`

## Related agents

- `truth-keeper` — drift between SoT and replicas
- `architect` — architecture file as SoT
- `declutter` (if present) — mechanical cleanup after rules are clear

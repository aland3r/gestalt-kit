# Kit audit

Systematic pass over every artifact under `gestalt-kit/` (skills, partials,
commands, agents) — shape correctness, staleness, broken links, orphans,
duplication, and drift against the live `portfolio.kit_docs` runtime copy.
`maestro` runs this; any agent can be asked to run a narrower slice of it.

## When to run

- Owner asks to "apurar"/audit skills, partials, commands (spells).
- Before a big kit reorg or rename (e.g. the spell→command rename this
  procedure would have caught in one pass instead of file-by-file).
- Periodically — rot accumulates quietly (a partial nobody links anymore, a
  skill referencing a path that moved).

## Scope per artifact type

| Type | Correct shape | Common rot |
|------|----------------|------------|
| **Agent** | `agents/<name>.md`, frontmatter `name`/`description`/`model`/`skills` | Description doesn't say when to use it; boundaries missing; roster (README + maestro.md) out of sync with actual files |
| **Skill** | `skills/<name>/SKILL.md` (thin pointer) + `reference.md` (full body) | Body pasted into `SKILL.md` instead of `reference.md`; `reference.md` missing |
| **Command** | Skill + `disable-model-invocation: true` in frontmatter | Missing `argument-hint`; still called "spell" in prose; not listed in README's Commands section |
| **Partial** | `partials/*.md`, linked (never pasted) from every skill/agent that uses it | A skill/agent pastes the partial's content inline instead of linking; partial exists but nothing links to it (orphan) |
| **Architecture** | `docs/architecture.md` only | A second "how the system is shaped" doc drifting from it |

## Procedure

1. **Enumerate** — `Glob` each folder: `agents/*.md`, `skills/*/SKILL.md`,
   `skills/*/reference.md`, `partials/*.md`. Note counts; compare to the
   README's own tables (agents roster, Commands section) — mismatch is
   itself a finding (README rot).
2. **Shape check** — for each agent/skill, read frontmatter. Flag: missing
   `description` trigger phrases, agents without a clear "not for X" boundary
   line, commands missing `disable-model-invocation: true`.
3. **Link check** — grep every `.md` under `gestalt-kit/` for markdown links
   (`](../...)`, `](./...)`) and verify the target path exists (`Read` or
   `Glob`). A broken relative link after a file move/rename is the single
   most common rot in this kit.
4. **Orphan check** — for each partial, grep the rest of the kit for a link
   to it. Zero incoming links → ask the owner whether it's still load-bearing
   or should be folded/deleted (`declutter`'s call once confirmed).
5. **Duplication check** — flag near-identical prose blocks across two
   agents/skills (a rule copy-pasted instead of linked as a partial) — this
   is exactly the case maestro's "prefer partial over pasting" teaching
   exists for. Hand off the actual merge to `declutter` or fix inline if
   trivial.
6. **Terminology check** — grep for stale names after a rename (this session
   found leftover "spell" in 15+ files after the concept had already been
   renamed in the primary doc). A rename isn't done until this grep is clean
   kit-wide, including `.cursor/skills/` adapters and `portfolio/` app code
   that mirrors kit vocabulary (`kit.kinds.*` i18n keys, `KIND_ORDER` arrays).
7. **Drift vs live DB** — `portfolio.kit_docs` is a synced copy of this
   folder (see `gestalt-kit/partials/kit-depara.md`). Compare counts/slugs;
   large drift → suggest `/kit-depara` rather than assuming either side is
   stale.

## Output shape

```text
Agents: N files — shape issues: … | roster drift: …
Skills: N (SKILL.md + reference.md pairs) — shape issues: …
Commands: N — missing disable-model-invocation: … | stale "spell" refs: …
Partials: N — orphans: … | pasted-not-linked: …
Broken links: … (file:line → missing target)
Duplication: … (candidates for a shared partial)
Kit↔DB drift: aligned | drift (run /kit-depara)
Action: fix now (list) | flag for owner (list) | hand to declutter (list)
```

## Boundaries

- Read-only audit — fixes to trivial issues (a broken link, a stale name)
  can happen inline; anything structural (merging two skills, deleting a
  partial) goes back to the owner or `declutter`.
- Does not audit product code correctness or UC content quality — that's
  `truth-keeper` (SoT) and `content-strategist` (description quality).
- Does not rename ORCA product vocabulary — that stays `ooux`'s call even
  when this audit surfaces a naming inconsistency in the kit's own terms.

## Related

- [maestro.md](../../agents/maestro.md) — owns this audit; "Kit structure
  review" / "Kit audit output" sections summarize the same procedure inline.
- [kit-depara.md](../../partials/kit-depara.md) — kit↔Supabase diff mechanics.
- `declutter` — structural cleanup once duplication/orphans are confirmed.

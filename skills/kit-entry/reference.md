# Kit entry — cold start without history

Use this when the session has **no prior context** or the owner asks “where do
I look?”. Follow the steps; do not guess from chat memory.

## Bootstrap sequence

1. **Hub entry** — read repo root `AGENTS.md` (table of paths).
2. **Kit home** — read `gestalt-kit/README.md` (skill / agent / partial / command).
3. **Navigation contract** — read
   [partials/kit-navigation.md](../../partials/kit-navigation.md) (parsing +
   adapter rules for every IDE).
4. **SoT matrix** — read [partials/sot-matrix.md](../../partials/sot-matrix.md).
5. **Scope** — read [partials/active-scope.md](../../partials/active-scope.md)
   (Portfolio + Deviante only; Flashbrix forbidden).
6. **Architecture** — skim `gestalt-kit/docs/architecture.md` § Folder plant +
   § Kit↔DB if touching agents/skills/site kit.
7. **Task layer** — load the relevant skill or agent from `gestalt-kit/skills/`
   or `gestalt-kit/agents/`.

Then proceed with the owner’s goal.

## Host-specific discovery (adapters only)

| Host | How skills appear | Where to edit |
|------|-------------------|---------------|
| Cursor | `.cursor/skills/<name>/SKILL.md` → links here | `gestalt-kit/skills/<name>/` |
| Claude Code | `--plugin-dir gestalt-kit` or plugin manifest | Same kit paths |
| Any | Open `c:\gestalt` at hub root | Same kit paths |

**Cursor `skills-cursor/` (global IDE skills)** — product features (hooks, PR
babysit, etc.). **Not** Gestalt domain knowledge. Do not merge or duplicate
Gestalt content there.

## Quick routing

| Question | Open |
|----------|------|
| Who should act? | `agents/maestro.md` |
| Kit vs Supabase depara? | `agents/truth-keeper.md` + `/kit-depara` |
| Which file is truth? | `agents/truth-keeper.md` + `partials/sot-matrix.md` |
| Delete duplicates? | `agents/declutter.md` |
| UC before coding? | `partials/uc-esteira.md` + command `/uc-gate` |
| DB / schema? | skill `gestalt-database` |
| Workspace map? | skill `gestalt-context` |
| Current AI quotas? | `partials/ai-tooling.md` |

## After reading

- Prefer **linking** partials over restating them.
- If two paths disagree → `truth-keeper` (SoT wins; adapters are never SoT).
- If `.cursor/skills` body grew beyond a thin pointer →
  `node scripts/sync-cursor-adapters.mjs`.

# UC Validate — orchestration mechanics

`/uc-validate` is a coordination layer, not a second gate. [uc-gate](../uc-gate/SKILL.md)
owns the explicit-yes write; this command owns cheaply fanning out to the
specialist agents so the owner sees one checklist instead of calling each
agent by hand.

## Why a separate command instead of folding this into /uc-gate

`/uc-gate` is deliberately minimal (load → card → confirm → mark) so the
owner's "sim" stays a single, unambiguous action. Bolting three agent
calls onto that same command would make the gate slower and muddier about
what exactly was confirmed. `/uc-validate` stays optional and composable
instead.

## Model routing (token economy)

| Layer | Model | Why |
|-------|-------|-----|
| This command's own coordination (read UC, decide which gates to run, aggregate) | Cheapest available (Haiku) | Pure text-shuffling — no judgment call being made here |
| `product-manager` | Sonnet (its own frontmatter) | Viability calls need real reasoning against the sprint plan |
| `architect` | Sonnet (its own frontmatter) | Pattern/anti-pattern judgment against `architecture.md` |
| `gamifier` | Haiku (its own frontmatter) | Audit/query work only — see [gamifier agent](../../agents/gamifier.md) |

Never override a specialist agent's configured model from this command —
the point is to stop paying Sonnet for coordination, not to cheapen the
judgment calls that actually need it.

## Launch pattern

Launch requested gates as **background Agent calls** (owner's explicit
choice, 21/07) rather than sequential inline calls:

```
Agent({
  description: "product-manager viability check — {abp_id}",
  subagent_type: "general-purpose"  // or a dedicated agent slot if the
                                      // harness exposes product-manager /
                                      // architect / gamifier as named types
  prompt: "... full context: UC id, live spec card fields already loaded,
            what to judge (viability vs sprint-plan-*.md) ...",
  run_in_background: true
})
```

Do the same for `architect` (pattern check against `docs/architecture.md`)
and, when the UC is already `accepted`, `gamifier` (quest/version sync
within its boundary — see [gamifier agent § Scope](../../agents/gamifier.md#scope-hard-boundary--owner-defined-2107)).

Each background call returns independently; do not fabricate or predict
a result before its notification arrives. Aggregate only what has
actually come back, and say so if something is still running when the
owner asks.

## Default gate set by moment

| When | Default `--gates` |
|------|--------------------|
| Before `/uc-gate` (spec not yet confirmed) | `product-manager,architect` |
| After implementation, before accept | `architect` (drift check) |
| After ACs verified (accept) | `gamifier` |

Owner can always override with an explicit `--gates` list, including
running all three at once.

## Checklist output shape

```text
UC: {abp_id} · esteira: {review_status} (live-checked at {time})
product-manager: {verdict} — {one-line why} (or "not requested")
architect: {verdict} — {one-line why} (or "not requested")
gamifier: {synced | blocked: reason} (or "not requested")
Blocking?: {yes/no — what, if yes}
Next: {/uc-gate | implement | fix X first}
```

## Boundaries

- Never writes `metadata.esteira` — that's `/uc-gate`'s job, after the
  owner's explicit "confirmado".
- Never invents a UC's spec content — load live, same rule as everywhere
  else in the esteira ([uc-esteira.md](../../partials/uc-esteira.md)).
- Does not replace the "Iteration approval" table in
  [uc-esteira.md](../../partials/uc-esteira.md#iteration-approval-end-of-each-iteration) —
  that's a batch-level sign-off across more agents (researcher,
  ui-designer, usability-tester, truth-keeper, content-strategist too),
  this command is per-UC and only the three gates listed above.

# UC Review — OOUX + UX Gatekeeping

Before a UC ships code, it passes two gates in sequence:

1. **OOUX Early Clarification** (`oouxer`) — Objects, Roles, CTAs, Attributes
   are real and match the Deviante ORCA Hub.
2. **UX Text Review** (`ux-writer`) — descriptions and steps read clearly,
   use OOUX vocabulary consistently, and meet quality bars.

Only after both pass does the UC move to `/uc-gate` (owner confirmado) and
then implementation.

## Phase 1 — OOUX Early Clarification

The `oouxer` agent:

1. Loads the Deviante ORCA Hub from Notion (Objects, Roles, CTAs, Attributes
   databases).
2. Reads the UC's description_what, steps, and ACs.
3. Asks the owner (stakeholder): what objects does this UC touch? Which roles
   act? What CTAs are primary? Any new attributes or relationships?
4. Proposes object cards (one per distinct object the UC mentions).
5. Cross-checks UI/Figma labels against the Hub if the UC has a surface yet.
6. Closes with: objects approved, open gaps (unmapped nouns), next step.

Output: **Object cards** (Object, Definition, Core attributes, Relationships,
Primary roles, Key CTAs, Open questions).

## Phase 2 — UX Text Review

The `ux-writer` agent:

1. Loads the OOUX outcomes from Phase 1.
2. Reads the UC's description_why/what/bounds, steps, and ACs.
3. Checks:
   - Does `description_why` answer "why does this UC exist?" (objective, not
     mission statement)
   - Does `description_what` describe the UC, not the domain object?
   - Do `description_bounds` start/end triggers make sense?
   - Do step labels use OOUX vocabulary (Objects, Roles, CTAs)?
   - Are ACs testable and specific?
4. Proposes text revisions (inline comments, rewrites if needed).
5. Flags drift: "this AC names an object the OOUX phase didn't clarify — gap
   or typo?"
6. Closes with: text approved, revisions needed, ready for gate.

Output: **Revision checklist** (what changed, why, any new OOUX gaps found).

## When to run

- **Before `/uc-gate`** (owner confirmado) — ensure objects and text are solid.
- **Per UC, in order** — UC3, then UC4, then UC5, etc.
- **One-shot** — after UC review passes, move to implementation; if the UC
  re-opens post-accept, revisit this command (don't rerun lightly).

## Boundaries

- `oouxer` does not implement anything; it facilitates and captures decisions.
- `ux-writer` does not do strategy or ORCA naming — those stay with
  `content-strategist` / `oouxer`.
- This command does not replace `/uc-gate` (owner explicit yes); it happens
  before.
- Gaps found here go back to OOUX Hub (Notion), not into UC text as workarounds.

## Related

- Agent `oouxer`: [gestalt-kit/agents/oouxer.md](../../agents/oouxer.md)
- Agent `ux-writer`: [gestalt-kit/agents/ux-writer.md](../../agents/ux-writer.md)
- Esteira gate: [gestalt-kit/partials/uc-esteira.md](../../partials/uc-esteira.md)

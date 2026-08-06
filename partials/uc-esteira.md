<!--
PARTIAL: UC owner-confirmation gate + multi-day esteira (conveyor).
Link from: maestro, truth-keeper, product-manager, persona-crafter,
implement-deviante-uc, write-use-case, gamifier, ship-quest, declutter,
sot-matrix, architecture.
Do not copy-paste — link this file.
-->

# UC esteira — owner confirmation gate

**Hard rule:** no agent may implement a use case (API/web/schema for that UC)
until the owner has **explicitly confirmed** the spec card for that UC in this
session (or a durable `metadata.esteira` mark proves prior confirmation and
the owner re-affirms scope has not changed).

Owner has **not** reviewed every UC yet. Work proceeds as a multi-day
**esteira** (conveyor): one UC at a time through load → card → confirm →
code → verify → gamifier.

## Source of truth

| Concern | SoT | Replica |
|---------|-----|---------|
| UC **intent + runtime** (title, Why/What/Bounds, steps, ACs, status) | **Live Supabase** `portfolio.use_cases` (+ `use_case_steps`, `requirements`) | `gestalt-kit/vault/**/user stories/` (Obsidian authoring), site `/cases` |
| Quest progress | Live `portfolio.quests` | HUD / bootstrap JSON |

- Prefer **Supabase MCP** (`execute_sql` / `list_tables`) to load and update.
- Site edits on `/cases` write the DB — that **wins** over a stale vault file.
- Vault is for Obsidian authoring and git history. **Never silently overwrite
  DB from vault** (or vault from DB) without owner direction.
- If vault ≠ DB → report **drift**, ask which wins (**default: DB**).
- **If Supabase MCP is not connected:** do not stop at "blocked." A **direct
  read-only fallback already exists** — see [gestalt-database § ad-hoc reads
  without MCP](../skills/gestalt-database/reference.md#ad-hoc-read-queries-without-mcp)
  (`scripts/diag-*.mjs` pattern, JDBC creds from `deviante/api/.env`). Use it
  to **load** the esteira / UC rows. Writes (`spec_confirmed`, etc.) still
  need owner's explicit yes in this session before anything is persisted —
  MCP or the same direct-connection pattern may both be used for that write,
  never invent specs from memory or claim the DB changed without running it.

## Durable esteira mark (no new column)

Reuse existing `portfolio.use_cases.metadata` jsonb. Do **not** invent a
migration unless the owner asks for a first-class column later.

```json
{
  "esteira": {
    "review_status": "unchecked",
    "spec_confirmed_at": null,
    "confirmed_by": null,
    "notes": null
  }
}
```

| `review_status` | Meaning |
|-----------------|---------|
| `unchecked` | Not yet confirmed on the esteira (default / missing key) |
| `spec_confirmed` | Owner said yes to the spec card — coding allowed |
| `implementing` | Optional — agent started implementation |
| `accepted` | ACs verified; ready for gamifier / ship |

Also keep the normal lifecycle `status`: `draft` | `ready` | `shipped` |
`deprecated` (product publish state — **not** a substitute for the gate).

**List next on esteira (MCP):**

```sql
SELECT abp_id, title, status,
       metadata->'esteira'->>'review_status' AS review_status,
       metadata->'esteira'->>'spec_confirmed_at' AS spec_confirmed_at
FROM portfolio.use_cases
WHERE product_code IN ('io', 'deviante')
  AND (
    metadata->'esteira'->>'review_status' IS NULL
    OR metadata->'esteira'->>'review_status' = 'unchecked'
  )
ORDER BY product_code, uc_number;
```

**Mark confirmed (after explicit owner yes):**

```sql
UPDATE portfolio.use_cases
SET metadata = jsonb_set(
      COALESCE(metadata, '{}'::jsonb),
      '{esteira}',
      jsonb_build_object(
        'review_status', 'spec_confirmed',
        'spec_confirmed_at', to_jsonb(now()::text),
        'confirmed_by', '"owner"'::jsonb
      ),
      true
    ),
    updated_at = now()
WHERE abp_id = :abp_id;
```

If MCP is missing, ask the owner to run the update on `/cases` or SQL editor
— do not pretend it was written.

## Process (every UC)

**Owner's preferred pacing:** two separate passes, not one UC pushed
end-to-end before the next starts. **Phase A** (steps 1–3 below — load,
description completeness/quality audit, spec card draft) runs across *many*
UCs in one sitting, fixing `why`/`bounds` clarity everywhere first. **Phase
B** (steps 4–6 — explicit confirm, `spec_confirmed` gate, implement) happens
later, per UC, once the owner is ready to validate specs. Don't assume a
description fix means the owner also wants the esteira gate flipped in the
same breath — ask, or read which phase the owner is in.

**Within Phase A, write freely as the dialogue moves** (owner, 21/07): the
owner does not need to say "confirmado" after every single field edit
during the description back-and-forth — engaging in the dialogue (giving a
correction, adding a fact, asking for a rewrite) **is** the direction to
write it. Write, then read back to verify it landed (still mandatory — see
"chat edit is binding" above) — but don't pause the conversation waiting
for an explicit yes on every sentence. The **gate that does need explicit
yes** is Phase B's `spec_confirmed` — never set that without it.

1. **Name** — Owner names a UC (`ABP-DV-UC4`, “próximo da esteira”, …) or
   agent proposes the next `unchecked` row from the query above.
   Optional: if time/quota/scope look tight, run **`product-manager`**
   viability vs live `gestalt-kit/plans/sprint-plan-*.md` before the card.
2. **Load from DB** — Prefer MCP. Join steps + requirements. If no MCP →
   use the `gestalt-database` diag-script fallback, or blocked / paste from
   `/cases`.
   - **Description completeness audit (`truth-keeper`)** — every UC needs
     all three description parts: `description_why`, `description_what`,
     `description_bounds` (see [write-use-case § Description](../skills/write-use-case/reference.md)).
     UCs authored before this 3-part convention was enforced may have
     `why`/`bounds` `NULL` with only `what` filled — check on every load, not
     just the first time. A missing field is a **gap to ask about**, never a
     silent draft-and-write: `truth-keeper` (or whoever's driving the card)
     may propose draft text, but it stays unconfirmed until the owner
     approves it — same write-then-confirm rule as step 4.
3. **Spec card** — Show (concise):

   ```text
   SPEC CARD — {abp_id} · {title}
   Status: {status} · Esteira: {review_status}
   Why: …
   What: …
   Bounds: …
   Actor / object: …
   Main flow: (step keys + one-line each)
   Alternates / errors: …
   ACs: (codes + titles)
   Personas: (linked or “missing → persona-crafter”)
   Quests that will move: (quest_id list for this uc_number / use_case_id)
   Drift vault↔DB: aligned | drift (detail) | vault N/A
   ```

4. **Confirm or edit** — Owner confirms, or edits. **An edit the owner
   states directly in chat is a binding instruction, not a note to
   self** — it is not "applied" just because it was typed. Same turn (or
   the very next step, never deferred silently): produce the write (MCP,
   owner-run SQL, or the `gestalt-database` diag-script fallback), get it
   persisted, and **read it back to confirm with the owner before moving on**
   to the next UC or the next step of this one. Prefer edits on site
   `/cases` or DB when the owner does them directly. If vault differs,
   report drift; **DB wins** unless owner says otherwise.
5. **Gate** — Only after **explicit** owner yes (“confirmado”, “pode
   implementar”, …) → set `metadata.esteira.review_status = spec_confirmed`
   → then implement (`implement-deviante-uc` or IO path).
6. **Done** — Verify against the **confirmed** ACs. Set esteira
   `accepted` when owner agrees verification passed.
7. **Gamifier** — Sync quest steps to the **valid scope of this UC** (see
   gamifier skill). Optionally `/ship-quest` for fine-grained quests.
   Setting UC `status` to `ready`/`shipped` auto-flips the `-spec` quest.

## Forbidden

- Treating an owner's chat-stated edit (title, why/what/bounds, AC, ...) as
  applied without writing it to DB and reading it back to confirm — text
  pasted or typed in chat is not persisted text until verified against
  the DB.
- Coding before owner yes on the spec card.
- Claiming DB/vault updated when MCP (or write) was unavailable.
- Treating sprint nicknames (`UC1-2a`) as the UC SoT — resolve to `abp_id`
  / `portfolio.quests.quest_id` via truth-keeper if unclear.
- Overwriting live DB rows from vault sync without owner direction when
  site/DB already differs.
- **Drafting alternative flows (1a, 2a, 1b, 2b, error paths 2.1/2.2, etc.) before the main path is confirmed by all six agents and code is implemented.**

## Alternative flows — blocked until main path closes

As of 21/07/2026, all alternative flows (1a, 2a, error paths 2.1, 2.2, etc.) 
have been removed from the spec cards for UC3–UC15. They remain **out-of-scope** 
until:

1. **All six agents confirm the main path:**
   - `architect` (feasibility, patterns)
   - `deviante-backend` (Kotlin/Exposed persistence)
   - `deviante-frontend` (React component plan, API calls)
   - `oouxer` (ORCA object/CTA naming)
   - `ux-writer` (copy quality, descriptions)
   - `truth-keeper` (DB/vault/plan alignment)
2. **Main path code is implemented** and acceptance criteria verified.
3. **Owner explicitly approves** "alternatives next" for that UC.

### When an edge case blocks the main path mid-sprint

If a real edge case emerges during implementation (e.g., "what if the CSV 
upload fails?" in UC4), **do not add it silently**:

- Propose to owner + architect: "UC4 main path assumes success — error 
  handling is alternative 2.1. Defer or land before 25/07?"
- If owner approves, treat the alternative as a **separate esteira gate**: 
  draft spec, run agent checkpoint, get `/uc-gate` confirm, then implement.
- Log the scope change in `gestalt-kit/plans/sprint-plan-2026-07.md` § 
  "Cronograma" (the plan is updated **live when scope changes are real**, 
  not after).

### Why this rule

1. **Spec clarity:** alternatives multiply fast; limiting to main path keeps 
   spec reviewable in one pass.
2. **Token economy:** agent checkpoints consume tokens — focus them on critical 
   paths (UC4 parse, UC12 drift) where alternatives are truly risky, not on 
   optional flows.
3. **Sprint velocity:** main paths deliver value; alternatives reduce surprise 
   failures. Deliver the value first, then the safety net.

## Iteration approval (end of each iteration)

An **iteration** = a batch of UCs pushed through one phase of
[implementation order](../plans/sprint-plan-2026-07.md#ordem-de-implementação-caminho-feliz-primeiro)
(e.g. "happy path across UC1–UC4"), not a single UC. Before calling an
iteration done, each relevant agent signs off **within their own domain** —
one agent's "looks fine" does not cover another's:

| Agent | Approves |
|-------|----------|
| `researcher` | The implemented UCs actually serve the personas' real goals/frustrations — a persona-fit read, not just that the flow exists |
| `ui-designer` | Static visual QA on every touched surface — contrast, spacing, glued controls (see [[autolayout-ux]] and persona Miguel, `vault/io/personas/Miguel.md`). A fix confirmed on **one** surface (e.g. `/kit`) does not clear others — check each independently |
| `usability-tester` | Structured checklist pass against the named persona (default Miguel for touch/spacing) on any surface with new or changed interactive controls |
| `architect` | No architecture/pattern drift introduced (`docs/architecture.md` Endorsed patterns) |
| `truth-keeper` | DB/vault/site alignment after the iteration's writes; no silent drift |
| `content-strategist` | UC description text quality still holds after implementation-driven edits |
| `product-manager` | Iteration stayed inside its declared scope (e.g. happy-path-only) and inside sprint time/quota |

**What this catches (owner's own example):** a filter row where buttons sit
glued edge-to-edge — `ui-designer` should never wave that through, because
Miguel (touch accessibility persona) cannot reliably tap one control
without mis-hitting its neighbor. That is exactly the class of defect this
gate exists to force a check for, on **every** surface, **every**
iteration — not just the one instance already fixed at `/kit`.

No agent's silence counts as approval. Each reports pass/fail with what it
checked, in that agent's own output shape.

## Who does what

| Role | Duty on the esteira |
|------|---------------------|
| `maestro` | Lineup includes esteira steps; teach “no code without yes” |
| `truth-keeper` | SoT = live DB; report vault↔DB drift; audit why/what/bounds completeness on every load, ask (don't guess) when a field is missing |
| `persona-crafter` | Load/link personas for the UC; workshop if missing |
| `content-strategist` | Audits the **quality** of why/what/bounds text already registered in DB (objective Why, clear start/end Bounds) — always paired with `truth-keeper` since UC text is DB-owned; query via MCP or the `gestalt-database` diag-script fallback when needed, never judge quality from a stale vault copy |
| `implement-deviante-uc` (and IO implement) | **Hard stop** until gate passes |
| `write-use-case` / `uc-scaffolder` | Authoring; after edits, sync path is owner-directed; gate still required before code |
| `gamifier` / `/ship-quest` | After accept: align quests/steps to confirmed UC scope |
| `/uc-gate` | Optional command to fire the checklist + mark confirmed |

## Owner cheat-sheet (tomorrow)

1. Open a session → ask maestro for **próximo da esteira** (or name a UC).
2. Or run **`/uc-gate ABP-DV-UCn`**.
3. Read the spec card → confirm or edit on `/cases`.
4. Say **confirmado** → agent implements.
5. On done → gamifier / `/ship-quest` as needed.

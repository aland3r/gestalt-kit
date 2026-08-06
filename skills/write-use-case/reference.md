# Write Use Case

Author or edit ABP use cases. **Runtime SoT** is live Supabase
`portfolio.use_cases` (+ steps, requirements). Markdown under
**`gestalt-kit/vault/`** is the Obsidian / git **authoring replica**.

> **Writing UCs for the PT-BR scientific report?** Follow
> **[report-ptbr.md](report-ptbr.md)** instead of the three-part English format
> below — it consolidates Why/What/Bounds into a single PT-BR `Descrição`,
> stores the row in PT-BR, and mirrors the validated UC1/UC2 tables. This
> reference.md still governs internal/vault English authoring.

Before **implementation**, every UC must pass the owner confirmation gate:
[partials/uc-esteira.md](../../partials/uc-esteira.md). Authoring alone does
not authorize coding.

## Before Writing

1. Read [terminology.md](../terminology/reference.md) for ID formats
2. Read [deviante-domain.md](../deviante-domain/reference.md) for vocabulary and relationships
2b. **Object definitions live in the ORCA Hub, not in the UC.** Before
   writing `description_what`, assume the object involved (Process,
   Activity, Event Log, ...) is already defined there — `ooux` owns that
   definition, UC text should not re-define it inline. If the object isn't
   defined yet, that's a gap: pull `ooux` in to define it in the Hub, don't
   improvise a definition here. See [partials/ooux-vocabulary.md](../../partials/ooux-vocabulary.md).
3. Prefer loading the existing row from **DB via MCP** if the UC already exists;
   also skim 1?2 vault examples under `gestalt-kit/vault/` (e.g. `io/user stories/`,
   `products/deviante/user stories/`)
4. Check the MOC for the product (`gestalt-kit/vault/io/Portfolio.md`,
   `gestalt-kit/vault/products/deviante/Deviante.md`) for `<<include>>` /
   `<<extend>>` links
5. If vault ? DB ? report drift; **DB wins** unless the owner says otherwise.
   Do not silently overwrite the live row from vault (or vault from DB).
6. Check `description_why` / `description_what` / `description_bounds` are
   all non-null — UCs written before the three-part convention may only
   have `what` filled. A missing field is a gap: ask the owner (a draft
   suggestion is fine to offer, but do not write it until they confirm).
   See [uc-esteira.md](../../partials/uc-esteira.md) step 2.

## File Template

```markdown
---
id: DV-UC{N}
written on: DD/MM/YYYY
---
[[Deviante]]

> [!NOTE] Description
> **Why:** [Purpose ? why this use case exists; value for the actor or product.]
> **What:** [General specifics ? who, what object, v1.0 scope, constraints. No implementation.]
> **Bounds:** Starts when [trigger / entry state]. Ends when [guaranteed outcome on success; or explicit non-outcomes.]

| | Use Case ID | APB-DV-UC0{N} |
| :--- | :--- | :--- |
| | **Use Case Name** | [Name] |
| | **Actor** | Manager |
| | **Object** | [Domain object] |
| | **Pre-Condition** | [Required state before trigger] |
| | **Post-Condition** | [Guaranteed state after success] |
| **Step:** | **Actor Trigger Action:** | **Black Box System Response:** |
| 1 | [What the Manager does] | [What the system does in response] |
| 2 | ... | ... |

### Extension Points
*   **[Name]:** [Optional branch to another UC]

### Included Use Cases
*   **UC{N} - [Name]:** [Why it is included]

### Acceptance Criteria
*   **DV-UC{N}-AC1 (Title):** [Testable, specific requirement]

## Database (portfolio schema)

Each file maps to (site `/cases` and MCP write the same tables):

| Markdown | Table |
|----------|--------|
| File + metadata table + description | `portfolio.use_cases` |
| Steps table | `portfolio.use_case_steps` |
| `### Acceptance Criteria` | `portfolio.requirements` |

Set `status` / `visibility` when ready to publish (`ready` + `public`).
Esteira gate mark lives in `metadata.esteira` ? see `uc-esteira.md`.

`node scripts/sync-vault.mjs` pushes vault ? DB only when the owner directs
a sync (and after resolving drift). Prefer editing on `/cases` when the live
row is already the working copy.
```

## Rules

### Description

Every use case description uses **three parts in this order**:

| Part | Label | Content |
|------|-------|---------|
| 1 | **Why** | Purpose ? why the UC exists; problem solved or value delivered |
| 2 | **What** | General specifics ? actor, object, v1.0 rules, product constraints |
| 3 | **Bounds** | Where it **starts** (entry trigger / pre-condition in plain language) and **ends** (post-condition outcome) |

Rules:
- Write as `**Why:**`, `**What:**`, `**Bounds:**` inside the `[!NOTE] Description` callout
- **Why** first ? never open with steps, AC, or implementation
- **Why quality bar:** objective, answers one question directly ? *"Why
  does this UC exist? What would be impossible in the system without it?"*
  That's a thinking prompt to test your draft against, **not a sentence
  template** ? don't open every Why with "Without X, Y can't happen." Vary
  the phrasing; the question is for you, not the reader.
- **What quality bar:** answers *"what IS this USE CASE"* ? its scope and
  action in one sentence (e.g. "This use case handles X into/from Y"),
  **not a definition of the domain object involved**. Object identity
  (what a Process/Activity/Event Log *is*) is `ooux`'s job via the ORCA
  Hub ? assume it's already defined there (step 2b above); don't redefine
  the object inline here. Not a running list of every business rule,
  downstream dependency, or CRUD capability either ? those belong in
  Steps / ACs / architecture notes.
- **Bounds** must name a clear start trigger and end state (success path);
  alternates/errors stay in the steps table
- **Bounds quality bar:** a reader must be able to point to where the UC
  **starts** and where it **ends** without re-reading the Steps table ? if
  they can't, the Bounds text is too vague
- **Human fluidity bar:** vary connectors — periods, "because", "so",
  commas, parentheses — instead of stringing every clause together with em
  dashes. Read it aloud: if it sounds like a list of clauses stitched with
  "—", rewrite as plain sentences. `ux-writer` (prose pass, paired with
  `content-strategist`'s quality audit) owns this check.
- **Vary the opening across UCs — no template phrase.** The quality bars
  above (Why/What/Bounds tests) are things to satisfy, not a sentence to
  copy-paste with the noun swapped. Don't open every What with "This use
  case handles..." — lead with the actor, the object, an action, a
  consequence, whatever fits that UC best. Same idea for Why/Bounds. If two
  UCs in a row read like the same mad-lib, rewrite one.
- Keep each part to 1?2 sentences unless the UC truly needs more
- Black-box in **What** ? no library names, table names, or API paths unless already established in a prior UC

### Character limits (calibrated to the validated UC1/UC2)

UC1 (Maintain user account) and UC2 (Maintain Process) are the **validated gold
standard** — mirror their concision. Soft caps:

| Field | Max chars |
|-------|-----------|
| `description_what` | ~160 |
| `description_why` | ~180 |
| `description_bounds` | ~360 |
| `summary` | ~320 |

Over a cap usually means you're explaining *how* in a description — move it to
the Steps (each step naming its CTA).

### Steps
- Use third person: "They select...", "It displays..."
- **Actor steps are interactive tasks with the system.** The actor *does*
  something concrete — clicks, selects, types, uploads, assigns, confirms,
  applies, opens, drags. **Never passive/observational verbs** ("reviews",
  "sees", "looks at", "checks", "examines"): the actor executes, the system
  responds. A step that reads like the actor is merely looking is wrong —
  rewrite it as the action taken, or fold the seen content into the system
  response.
- **Name the control the actor activates** — the CTA/affordance that enables
  the task (e.g. "The Manager triggers the *Confirm mapping* CTA"), not a vague
  verb like "reviews". Each step stays traceable to a concrete interface
  element — deliberately good for HCI argumentation / PIBITI thesis evidence.
- Main flow: steps 1, 2, 3...
- Alternates (per step): 1a, 2a for edit/update paths; 1b, 2b for delete paths
- Full alternative flow (whole-scenario branch): A1, A2, A3...
- Errors: 2.1, 2.2 under the triggering step
- Black-box only ? no implementation details

### Relationships
- `<<include>>` ? `### Included Use Cases` (mandatory sub-flow)
- `<<extend>>` ? `### Extension Points` (optional branch from this UC or to another UC)
- **UML diagram as modeling reference:** when the owner shares a UML use-case
  diagram (image), it is the **system modeling reference**, not an
  illustration ? extract every `<<include>>`/`<<extend>>` link from it and
  read the extraction back to the owner for confirmation before writing
  anywhere (DB write needs `portfolio.use_case_relationships` to exist first
  ? see [gestalt-database](../gestalt-database/reference.md); until then,
  keep the confirmed reading in this session / the plan, don't invent a
  markdown-only home for it).

### Acceptance Criteria
- Format: `DV-UC{N}-AC{M} (Short Title):` followed by requirement
- Each AC must be independently testable
- Cover validation, edge cases, and data integrity

### Cross-References
- Reference other UCs by number and name: `UC5 - Resolve Mapping`
- Add `[[Deviante]]` wikilink at top (after frontmatter or before description)

## Quality Checklist

- [ ] File name matches `ABP-DV-UC{N}-{PascalCaseName}.md`
- [ ] Frontmatter `id` matches `DV-UC{N}`
- [ ] Use Case ID in table is zero-padded (`APB-DV-UC05`)
- [ ] Actor matches product: `Manager` (Deviante), `Visitor` (Portfolio public flows)
- [ ] Description has **Why ? What ? Bounds** in that order
- [ ] **Bounds** states where the UC starts and ends
- [ ] Included/Extension sections match the UML diagram
- [ ] At least 2 acceptance criteria
- [ ] No implementation leakage in steps (unless inherited from UC4 parser reference)
- [ ] If implementing next: esteira gate still required (`/uc-gate` or maestro)

## Reference Examples

Best examples to mirror (paths under `gestalt-kit/vault/`):
- `io/user stories/ABP-IO-UC1-DownloadCV.md` ? short UC; Why/What/Bounds
- `products/deviante/user stories/ABP-DV-UC2-MaintainProcess.md` ? CRUD with extension points
- `products/deviante/user stories/ABP-DV-UC3-MaintainActivity.md` ? technical rationale in description
- `products/deviante/user stories/ABP-DV-UC4-UploadEventLog.md` ? included use cases

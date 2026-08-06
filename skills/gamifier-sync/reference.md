# Gamifier Sync & Audit

**Purpose:** Quest log lives in `portfolio.quests` (Supabase) but gets out of sync with UC specs, sprint roadmap, and truth-keeper findings. This command detects and fixes those divergences.

**Owner use case:** You see a quest label "UC UC1 → API JDBC → Supabase Postgres" and think: "Is that still right? Did UC1 spec change? Does the roadmap still say UC1 is next?" This command answers those questions.

---

## Data sources (SoT map)

| Source | Table / File | What it holds | Role |
|--------|---|---|---|
| **Supabase (DB)** | `portfolio.quests` | Quest status, linked UC, progress | Live state |
| **Supabase (DB)** | `portfolio.use_cases` | UC spec, esteira review status | UC source of truth |
| **Repo (git)** | `gestalt-kit/plans/sprint-plan-2026-07.md` | Roadmap days, phases, priority order | Owner's intent |
| **Truth-keeper** | Findings log (chat/vault) | Drift between repo/DB, stale content | Auditor |

---

## Audit protocol (every sync)

### 1. Load quest row

```sql
SELECT quest_id, status, product_code, linked_uc, 
       description_what, progress_pct, updated_at
FROM portfolio.quests
WHERE quest_id = $0 OR ($0 IS NULL);  -- all if empty
```

For each quest, verify:
- ✓ `quest_id` matches naming convention (e.g., `UC1-1a`, `UC1-happy-path-intro`)
- ✓ `product_code` is in active scope (IO, DV; not MB, HA, FB)
- ✓ `status` is one of: `planned` | `active` | `in_progress` | `paused` | `done`
- ✓ `linked_uc` (if present) matches a real UC in `portfolio.use_cases`

**Flags:**
- ⚠ Quest with no `product_code` → orphan
- ⚠ Quest linked to UC that no longer exists → ghost
- ⚠ Quest status = `active` but spec says `unchecked` → mismatch

### 2. Load linked UC spec (if quest.linked_uc is not null)

```sql
SELECT abp_id, status, 
       metadata -> 'esteira' -> 'review_status' as esteira_status,
       body_md
FROM portfolio.use_cases
WHERE abp_id = quest.linked_uc;
```

Cross-check:
- ✓ UC `review_status` ∈ {`unchecked`, `spec_confirmed`, `impl_ready`, `shipped`, `ready`}
- ✓ Quest status aligns with UC esteira state:
  - If UC = `unchecked` → quest should be `planned` (not `active`)
  - If UC = `spec_confirmed` → quest can be `active` or `in_progress`
  - If UC = `shipped` → quest should be `done`

**Flags:**
- ⚠ Quest `active` but UC `unchecked` → out of sync
- ⚠ Quest `done` but UC not `shipped` → inconsistent
- ⚠ UC changed spec (body_md updated recently) → quest description may be stale

### 3. Cross-check sprint roadmap

Read `gestalt-kit/plans/sprint-plan-2026-07.md`:

**Roadmap table (days 22–30):**
```
| Day | Foco | ...
| 22/07 | Gate real da Fase 1 | UC3/UC4 spec + apply schema
| 23/07 | UC3 + UC4 happy path | ...
| ...
```

For each quest:
- ✓ If linked to UC1 or UC2 → should be `done` by 22/07 (already approved)
- ✓ If linked to UC3–UC13 → roadmap day ≤ today (or future if not started)
- ✓ If quest is `active` → roadmap should list that UC/quest for today or recent past
- ✓ Priority order matches roadmap (UC12 protected > UC9/10/11 ceeded)

**Flags:**
- ⚠ Quest is `in_progress` for UC that roadmap pushed to "hold until Fase X"
- ⚠ Quest is `done` but roadmap shows it as future (timeline corruption)
- ⚠ Quest for UC9/10/11 is `active` while UC12 is still `planned` (priority violated)

### 4. Consult truth-keeper findings

(No live API; check what truth-keeper has logged in recent chat / vault.)

Ask truth-keeper to report:
- Any drift between `portfolio.quests` and repo SoT (e.g., quest_id appears in quest log but not in CLAUDE.md gamifier section)?
- Any orphaned quests (linked_uc deleted)?
- Any stale descriptions (quest copy doesn't match current UC spec)?

### 5. Build status matrix

```
Quest | Product | Linked UC | Quest Status | UC Esteira | Roadmap Day | Drift? | Action
------|---------|-----------|--------------|------------|-------------|--------|--------
UC1-happy-intro | IO | UC1 | done | spec_confirmed | 22/07 (past) | ✗ | OK
UC3-1a | DV | UC3 | active | unchecked | 23/07 | ⚠ UC unchecked! | Fix: pause quest until UC confirmed
UC9-alt-flow | DV | UC9 | in_progress | spec_confirmed | 29/07 | ⚠ Priority | Check: UC12 done? If not, UC9 should yield.
```

### 6. Propose next step

**If no drift:**
```
✓ Quest log coherent with Supabase + roadmap.
→ Next step per roadmap: UC3 spec confirmation (22/07 gate).
   Quest to activate: UC3-1a (Request access).
```

**If drift detected:**
```
⚠ 2 quests out of sync:
  - UC3-1a linked to UC3 (unchecked) but quest is "active" → should be "planned"
  - UC9-alt-flow (in_progress) but UC12 still "planned" → UC9 should yield

Resolution options (ask owner):
  1. Pause UC9-alt-flow, unblock UC12 (respect roadmap priority)
  2. Approve UC3 spec immediately (unblock UC3-1a quest activation)
  3. Update roadmap if priorities changed (owner decision)
```

---

## Output format (before --fix)

```
╔═══════════════════════════════════════════════════════════╗
║ GAMIFIER SYNC AUDIT                                       ║
╚═══════════════════════════════════════════════════════════╝

SOURCE SCAN:
  portfolio.quests (Supabase): 64 rows
  portfolio.use_cases (Supabase): 15 rows (UC1–UC15)
  Sprint roadmap: days 22–30 (Fase 1–6)
  Truth-keeper: last report 21/07 (no recent drift logged)

STATUS MATRIX:
┌──────────────┬─────────┬──────────┬─────────────┬──────────┬────────────┬────────┐
│ Quest        │ Product │ Linked   │ Quest       │ UC       │ Roadmap    │ Drift? │
├──────────────┼─────────┼──────────┼─────────────┼──────────┼────────────┼────────┤
│ UC1-1a       │ IO      │ UC1      │ done        │ shipped  │ 22/07 ✓    │ ✗ OK   │
│ UC3-1a       │ DV      │ UC3      │ active ⚠    │ unchecked│ 23/07      │ ⚠ WARN │
│ UC9-alt-flow │ DV      │ UC9      │ in_progress │ s.c.     │ 29/07      │ ⚠ WARN │
└──────────────┴─────────┴──────────┴─────────────┴──────────┴────────────┴────────┘

FINDINGS:
  ✗ 2 misalignments (quests out of sync with UC esteira)
  ✗ 1 priority violation (UC9 active before UC12 done)

RECOMMENDATIONS:
  1. UC3-1a: Pause quest until UC3 review_status → spec_confirmed (22/07 gate)
  2. UC9-alt-flow: Yield to UC12 (roadmap priority)
     If UC12 blocked, escalate to product-manager

NEXT STEP (roadmap coherent):
  → Day 22/07: UC3 spec gate + apply schema
  → Activate: UC3-1a (Request access catalog)
  → Protect: UC12 (drift ADWIN — no other work competes)

With --fix:
  - Pause UC3-1a (status → planned)
  - Pause UC9-alt-flow (status → planned)
  - Keep UC1-1a/UC2-1a/UC2-1b at 'done'
  - Confirm next active quest = UC3-1a (after 22/07 gate)
```

### Output (after --fix)

```
SYNC APPLIED:
  ✓ Paused 2 quests (UC3-1a, UC9-alt-flow)
  ✓ Confirmed UC1–UC2 status = done
  ✓ Updated quest descriptions (3 stale → current spec)
  ✓ Verified priority order (UC12 protected)

NEW STATE:
  Active: UC1-1a (done), UC2-1b (done)
  Next to activate: UC3-1a (awaiting 22/07 spec gate)

Roadmap alignment:
  ✓ Quest log now coherent with sprint-plan-2026-07.md
  ✓ Truth-keeper has no new drift to report
```

---

## Integration with other agents

### product-manager
- Reviews recommendations before `--fix`
- Approves if priorities changed (roadmap vs quests diverged)
- Escalates to owner if UC gate slipped

### architect
- Verifies UC spec hasn't changed in incompatible ways (quest description stays relevant)
- Flags if schema or UC dependency changed (operations, activities, etc.)

### truth-keeper
- Consulted for drift report (at start of audit)
- Updated after sync (new findings logged if new orphans/ghosts created)

### gamifier (self)
- Runs as routine: `/gamifier-sync --routine` (e.g., daily check)
- Runs on-demand: `/gamifier-sync [quest-id]` (owner requests verification)
- Runs before ship: `/gamifier-sync --pre-ship` (before `/ship-quest`)

---

## Commands

### Audit all quests (no changes)
```bash
/gamifier-sync
```
→ Scan all 64 quests, report drift, propose next step

### Audit one quest
```bash
/gamifier-sync UC3-1a
```
→ Focus on UC3-1a: is it coherent with UC3 spec + roadmap?

### Audit + fix (ask for approval)
```bash
/gamifier-sync --fix
```
→ Propose fixes, wait for yes/no, apply if approved

### Routine background check
```bash
/gamifier-sync --routine
```
→ Daily: silent if coherent, alert if drift found

### Pre-ship verification
```bash
/gamifier-sync --pre-ship
```
→ Before `/ship-quest`, verify that quest's linked UC is ready (not orphaned, not ghost)

---

## Related

- [gamifier.md](../../agents/gamifier.md) — agent that manages quests
- [sprint-plan-2026-07.md](../../plans/sprint-plan-2026-07.md) — roadmap SoT
- [uc-esteira.md](../../partials/uc-esteira.md) — UC approval gate
- [sot-matrix.md](../../partials/sot-matrix.md) — which data lives where

---
name: mpsbr-certifier
description: >-
  Certify software process maturity against MPS.BR levels (G–A). Assess evidence,
  determine current level, recommend path to next. Deviante primary use case.
disable-model-invocation: false
argument-hint: "[process-name]"
---

# MPS.BR Process Certifier

Certify the maturity level of a software process against MPS.BR (Brazilian maturity model).

**Full reference:** [gestalt-kit/partials/mpsbr-levels.md](../../partials/mpsbr-levels.md)

---

## How to Use

### Command (via QA agent)

```
qa: Certify the maturity of "UC4 Event Log Upload" process
```

OR

```
qa: What level is the "Top Nav Responsive Design" process at? Certify + recommend next steps.
```

### Input

- **Process name** (e.g., "UC4 Event Log Upload", "Login authentication")
- **Optional:** specific MPS.BR level to audit against (default: auto-detect from G→D)

### Output

```
Process: [name]
Current Level: [G|F|E|D|C|B|A]
Evidence: [bullets for why this level]
Gaps: [what's missing for next level]
Recommendation: [actions + effort estimate]
```

---

## Certification Protocol

### Step 1: Identify Process Scope

Which **7 process areas** apply (G–D range)?
- GPR (Project Management)
- GRE (Requirements)
- GCI (Quality Assurance)
- GSC (Configuration Management)
- AMA (Acquisition — if vendor-related)
- MED (Measurement)
- OAT (Environment)

Example: UC4 → GPR + GRE + GCI + GSC + MED (not AMA/OAT)

### Step 2: Collect Evidence

For each process area, search the codebase + repo for:
- **Level G evidence:** task lists, deliverable names, basic quality checks
- **Level F evidence:** formal plans, requirements docs, risk registers, metrics
- **Level E evidence:** org-wide process templates, post-project reviews, policy
- **Level D evidence:** quantitative goals, baselines, control charts, causal analysis

Evidence sources:
- Git commits + PR descriptions
- Sprint plans (quest docs)
- Requirements specs (UC cards in portfolio.use_cases or vault/)
- Code review records (PR comments)
- Defect tracking (issues, TODOs)
- Metrics dashboards (if any)
- Process docs (gestalt-kit/docs/*, gestalt-kit/partials/*)

### Step 3: Determine Current Level

**Start at G, move up until you find UNMET criteria:**

- **G?** Do all 7 areas have basic evidence? If NO → cannot certify.
- **F?** Is there formal planning, execution, tracking? If NO → max level is G.
- **E?** Is process documented org-wide + reusable? If NO → max level is F.
- **D?** Are quantitative goals + baselines + monitoring in place? If NO → max level is E.
- **C+?** (deferred — not active yet)

**Certified level = highest level where ALL criteria are met.**

### Step 4: Document Gaps

For next level up, list what's missing:
- Specific artifacts (docs, metrics, reviews)
- Process changes (e.g., "establish SLA", "post-project review")
- Effort estimate (hours/days)

### Step 5: Recommend Path

"To reach level [X], implement: [actions]. Effort: ~[time]. Owner: [QA agent | architect | product-manager]."

---

## MPS.BR Level Summary

| Level | Focus | Key Artifact |
|-------|-------|--------------|
| **G** | Basic execution | Task list + deliverables |
| **F** | Formal planning + tracking | Project plan + metrics |
| **E** | Org-wide process | Process template + tailoring |
| **D** | Quantitative control | Baseline + SLA + control chart |
| **C** | Quantitative optimization | Performance optimization cycle |
| **B** | Systematic innovation | Innovation backlog + measured improvements |
| **A** | Continuous improvement | Predictive capability + proactive optimization |

---

## Example: UC4 Certification

**Input:** "Certify UC4 Event Log Upload"

**Analysis:**

G Criteria (7 areas):
- ✅ GPR: Quest plan exists (gestalt-kit/plans/UC4-EVENT-LOG-UPLOAD-QUEST.md)
- ✅ GRE: Requirements in quest breakdown (A1/A2/B1/B2/B3)
- ✅ GCI: Quality gates defined in quest (testing, staging preview)
- ✅ GSC: Git repo + commits (deviante/api, deviante/web)
- ✅ MED: Metrics identified (coverage, ship timeline)
- ✅ OAT: Tools named (Ktor, React, Tailwind, Vercel)
- ✅ AMA: Not applicable (no external vendors)

F Criteria (formal planning):
- ✅ Project plan: Sprint kickoff doc (timeline, checklists, contact matrix)
- ✅ Requirements: UC4 quest breakdown (formal, versioned)
- ✅ Risk mgmt: Blockers identified (B3 top nav)
- ✅ Tracking: Progress vs. quest checklist
- ✅ Metrics: Time tracking (23–24/07 window)
- ⚠️ Code review: Planned but not yet done (gates UC4 ship)

E Criteria (org-wide process):
- ✅ Org template: gestalt-kit/plans/SPRINT-KICKOFF... follows org pattern
- ✅ Tailoring: UC4 is adaptation of org quest template
- ⚠️ Post-project review: Not scheduled yet
- ❌ Quality policy: General org policy exists, not UC4-specific

**Certification:** **F (Gerenciado)**

**Gaps to E:**
- Define UC4-specific quality policy (acceptance criteria by component)
- Schedule post-project review (24/07 EOD or 25/07)
- Document how UC4 tailored org template

**Effort to E:** ~4 hours (one afternoon)

**Recommendation:** Reach E by **25/07 EOD** (still in sprint). QA agent owns post-project review; architect documents tailoring; product-manager approves quality policy.

---

## Certification Report Template

Use this format for formal certification:

```
┌────────────────────────────────────────────┐
│ MPS.BR Process Certification Report        │
├────────────────────────────────────────────┤
│ Process Name:  [UC4 Event Log Upload]      │
│ Model:         MPS.BR v3                   │
│ Assessed By:   QA Agent                    │
│ Date:          [2026-07-23]                │
│ Confidence:    [High/Medium/Low]           │
├────────────────────────────────────────────┤
│ CERTIFIED LEVEL: F (Gerenciado)            │
├────────────────────────────────────────────┤
│ EVIDENCE                                   │
│ Level G: All 7 process areas met ✅        │
│ Level F: Formal planning + tracking ✅     │
│ Level E: Org process doc + tailoring ⚠️   │
│ Level D: Quantitative goals ❌             │
├────────────────────────────────────────────┤
│ GAPS (to reach Level E)                    │
│ • Quality policy (UC4-specific)            │
│ • Post-project review (schedule)           │
│ • Tailoring doc (org template → UC4)       │
├────────────────────────────────────────────┤
│ NEXT STEPS                                 │
│ 1. QA agent: schedule post-proj review     │
│ 2. Architect: document tailoring           │
│ 3. Product-mgr: approve quality policy     │
│ Effort: ~4 hours | Timeline: 25/07 EOD     │
└────────────────────────────────────────────┘
```

---

## Related

- [qa.md](../../agents/qa.md) — QA agent (invokes this skill)
- [mpsbr-levels.md](../../partials/mpsbr-levels.md) — Level definitions + checklist
- [quality-gates.md](../quality-gates/reference.md) — Gate enforcement

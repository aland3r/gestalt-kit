# MPS.BR Certification Reference

**Full MPS.BR documentation:** [gestalt-kit/partials/mpsbr-levels.md](../../partials/mpsbr-levels.md)

This file provides the procedural reference for running certifications.

---

## Invocation

```
/qa certify [process-name]
```

OR (via agent request):

```
qa: Assess and certify "UC1 Login" process against MPS.BR
```

---

## Certification Workflow

### Phase 1: Intake

**Input:**
- Process name (e.g., "UC4 Event Log Upload")
- Optional: specific level to audit (default: auto from G→D)
- Optional: scope (entire process vs. subcomponent)

**Clarify:**
- Which MPS.BR process areas apply? (GPR, GRE, GCI, etc.)
- What's the scope boundary? (just backend, or full UC including tests + docs?)
- Who should approve the certification? (product-manager, architect)

---

### Phase 2: Evidence Collection

**Grep/Read the codebase for:**

#### Level G Evidence
- [ ] Task list or quest plan (gestalt-kit/plans/*.md)
- [ ] Deliverable names (requirements, code files, tests)
- [ ] Quality checklist (even informal: "must pass tests", "mobile responsive")
- [ ] Version control (git commits exist for this process)
- [ ] Baseline metadata (tool inventory, team roster)

#### Level F Evidence
- [ ] Project plan document (timeline, milestones, resources)
- [ ] Requirements specification (formal: UC card, quest breakdown, or spec doc)
- [ ] Risk register (identified risks, severity, mitigation)
- [ ] Code review records (PR comments, PR approvals)
- [ ] Defect tracking (if any found: issues, TODOs, notes)
- [ ] Metrics collected (coverage %, cycle time, burndown, test results)

#### Level E Evidence
- [ ] Org-wide process template document (gestalt-kit/docs/*, gestalt-kit/plans/*)
- [ ] Tailoring record (how THIS process adapted the template)
- [ ] Post-project review notes or scheduled review
- [ ] Org quality policy (documented, not just implicit)
- [ ] Training materials or onboarding docs for the process

#### Level D Evidence
- [ ] Quantitative goals (SLA, defect density, release date, perf target)
- [ ] Baseline metrics (historical data from prior similar processes)
- [ ] Process performance monitoring (tracked weekly/daily against baseline)
- [ ] Control charts or dashboards (showing variance detection)
- [ ] Root cause analysis records (when variance detected, what was found)
- [ ] Corrective/preventive action log (what fixed each issue)

**Search locations:**
- `gestalt-kit/plans/` — sprint plans, quest docs, checklists
- `gestalt-kit/docs/` — architecture, process templates, standards
- `gestalt-kit/partials/` — org policies, partial standards
- Git history — PR descriptions, code review comments
- `portfolio.use_cases` (Supabase) — UC spec + metadata
- Live dashboards (if Vercel, GitHub, Supabase have metrics)

---

### Phase 3: Level Determination

**Algorithm:**

```
level = G
if all_g_evidence.met:
  level = F
  if all_f_evidence.met:
    level = E
    if all_e_evidence.met:
      level = D
      if all_d_evidence.met:
        level = C  # (deferred)

return level
```

**Key rule:** Certify the HIGHEST level where **ALL** criteria are met.
- If F is missing one artifact (e.g., risk register), max certification is G.
- If E is missing one artifact (e.g., org quality policy), max certification is F.

---

### Phase 4: Gaps & Recommendations

**For each level above current, document:**

| Gap | Artifact | Owner | Effort | Timeline |
|-----|----------|-------|--------|----------|
| Quality policy undefined | Formal doc (1 page) | Product-manager | 1 hour | This week |
| Post-project review missing | Meeting notes + action items | QA agent | 2 hours | 24 hrs |
| SLA not quantified | Target values (document) | Architect | 0.5 hours | This week |

**Effort estimate:** Sum hours for all gaps to next level.

**Timeline:** Recommend realistic date (e.g., "25/07 EOD" if doable in 4 hours this sprint).

**Owner:** Assign to architect, QA agent, or product-manager based on artifact type.

---

### Phase 5: Certification Report

**Output format (use template from SKILL.md):**

```
Process: [Name]
Current Level: [G|F|E|D]
Evidence: [summarize what proved this level]
Gaps to [Next Level]: [list artifacts + effort]
Recommendation: [actions + owner + timeline]
```

**Confidence level:**
- **High:** Evidence is recent, documented, verifiable in code/git
- **Medium:** Some evidence is informal or inferred from code patterns
- **Low:** Minimal evidence; relies on team interviews or assumptions

**Approval:** Requires sign-off from product-manager or architect.

---

## Examples

### Example 1: UC1 Login (Complete)

**Evidence collected:**
- G: Quest plan, requirements in portfolio.use_cases, code in git, basic QA checklist
- F: Sprint plan (SPRINT-KICKOFF doc), PR review, tests passing, coverage >80%
- E: Org quest template used (gestalt-kit/plans/UC-TEMPLATE.md), post-project review scheduled
- D: SLA defined (login <500ms), baseline from similar UCs, monitoring via Vercel metrics

**Certification:** **E (Parcialmente Definido)**

**Gaps to D:**
- Root cause analysis log (none yet)
- Corrective action tracker (not formalized)
- Effort: 2 hours

**Recommendation:** "Reach D by 25/07. Architect creates action log template; QA agent opens first entry if any perf issue surfaces during testing."

---

### Example 2: UC4 Event Log Upload (In Progress)

**Evidence collected:**
- G: Quest plan exists, A1/A2/B1/B2/B3 breakdown, version control ready
- F: SPRINT-KICKOFF doc (timeline + checklists), requirements (quest breakdown), risks (B3 blocker)
- E: Org template adapted (sprint-kickoff is standard), post-project review NOT YET SCHEDULED
- D: No quantitative goals yet (no SLA defined)

**Certification:** **F (Gerenciado)**

**Gaps to E:**
- Post-project review (schedule meeting, capture notes)
- Org quality policy for UC4 (1-page document: acceptance criteria)
- Tailoring document (how sprint-kickoff was customized for UC4)
- Effort: 4 hours

**Recommendation:** "Reach E by 25/07 EOD. Schedule post-review for 24/07 PM. Architect + product-manager write policy doc + tailoring note. All items fit in current sprint."

---

### Example 3: Top Nav Responsive Fix (Minimal)

**Evidence collected:**
- G: Task in UC4 quest (B3), responsive testing checklist defined
- F: No formal plan or requirements doc (part of larger UC4, not standalone)
- E: No org process for design review (ad hoc currently)
- D: No quantitative design goals (perf budget, accessibility score)

**Certification:** **G (Parcialmente Gerenciado)**

**Gaps to F:**
- Design requirements document (acceptance criteria by viewport)
- Design review checklist (formal, not just quest notes)
- Accessibility metrics (target score: WCAG 2.1 AA)
- Effort: 3 hours

**Recommendation:** "Reach F by 23/07 EOD (same day as UC4 ship). Designer creates requirements doc + checklist. QA agent reviews before ship gate."

---

## Certification Governance

### Who Can Certify?

- **QA agent** — primary authority (runs this skill)
- **Architect** — secondary review (architecture alignment)
- **Product-manager** — approval gate (business requirement: process maturity aligns with sprint goals)

### When to Certify?

- **End of sprint** (post-project review) — every process should be assessed
- **Gate before ship** (optional) — if MPS.BR level is a quality gate
- **Promotion to next org level** (e.g., G→F for the whole portfolio) — when many processes advance together

### Certification Validity

- **Valid for:** Until next sprint or material change to process
- **Expires:** If process changes (new tools, new team, new scope) — recertify
- **Appeal:** Product-manager can request re-assessment if they disagree

---

## Troubleshooting

### Gap: "No evidence for Level F"

**Likely cause:** Process lacks formal documentation (implicit understanding only).

**Remedy:** Create project plan document (1–2 pages) + requirements spec.

### Gap: "Can't find metrics"

**Check sources:**
- GitHub PR metrics (review time, CI results)
- Vercel deployments (build time, uptime)
- Supabase query logs (response time)
- Manual tracking (spreadsheet, confluence notes)

If none exist: start with one metric (e.g., cycle time or defect count).

### Gap: "Post-project review scheduled but not done yet"

**Decision:** 
- If review is scheduled <1 week away → certify current level, note review pending
- If review is >1 week away → cannot certify E yet (gap too large)

### Gap: "Quantitative goals exist but baseline doesn't"

**Decision:** Can certify D only if baseline is collected or estimated from prior data.

If no baseline: certify F (goals exist but not yet monitored); set 1-week deadline to collect baseline.

---

## Related

- [qa.md](../../agents/qa.md) — QA agent (runs this skill)
- [mpsbr-levels.md](../../partials/mpsbr-levels.md) — MPS.BR level definitions
- [quality-gates.md](../quality-gates/reference.md) — Enforcement at each level

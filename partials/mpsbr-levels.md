# MPS.BR v3 — Maturity Levels (Deviante Reference)

**Model:** MPS (Modelo de Maturidade do Processo de Software) — Brazilian standard  
**Version:** v3.0 (2016)  
**Scope:** 7 process areas in G–D range; additional areas in E–C–B–A (not active for Deviante yet)

---

## Level Hierarchy

```
A (Otimizado)           ← Continuous innovation + predictive
B (Otimizado)           ← Systematic innovation + quantitative optimization
C (Gerenc. Quantitativo)← Quantitative control (SLA, metrics, thresholds)
D (Definido)            ← Process standardized org-wide (documented, repeatable)
E (Parc. Definido)      ← Project process documented (not yet org-wide standard)
F (Gerenciado)          ← Basic planning + execution + tracking
G (Parc. Gerenciado)    ← Tasks executed, minimal formality
```

---

## Level G (Parcialmente Gerenciado)

**Goal:** Establish basic task execution and deliverable identification

**7 Process Areas:**
1. **GPR** — Gerenciamento de Projeto (Project Management)
   - Tasks planned at high level
   - Stakeholders identified
   - Deliverables named
   
2. **GRE** — Gerenciamento de Requisitos (Requirements Management)
   - Requirements captured (may be informal)
   - Changes tracked (basic log)
   
3. **GCI** — Garantia da Qualidade (Quality Assurance)
   - Quality criteria identified
   - Work products checked against criteria
   
4. **GSC** — Gestão de Configuração (Configuration Management)
   - Baselines identified
   - Changes controlled (via version control or naming)
   
5. **AMA** — Aquisição (Acquisition)
   - Vendor selection criteria defined
   - Contracts reviewed
   
6. **MED** — Medição (Measurement)
   - Metrics defined (may not be collected yet)
   - Goals stated
   
7. **OAT** — Organização do Ambiente (Environment Organization)
   - Tools selected
   - Infrastructure identified

**Evidence to Collect:**
- Task list (even informal)
- Deliverable names + descriptions
- Basic quality checklist
- Version control usage (git commits exist)
- Vendor list (if applicable)
- Tool inventory

**Certification Criteria:** All 7 areas have SOME evidence of task/deliverable management.

---

## Level F (Gerenciado)

**Goal:** Establish formal planning, execution, and tracking

**Superset of G + new requirements:**

1. **GPR** (expanded)
   - Project plan documented
   - Schedule established (with milestones)
   - Budget/resources allocated
   - Progress tracked against plan
   - Risks managed (identified, assessed)
   
2. **GRE** (expanded)
   - Requirements formally documented
   - Traceability to tests/code established
   - Change control process defined
   - Requirements review done
   
3. **GCI** (expanded)
   - Quality plan created
   - Reviews scheduled (code review, inspection)
   - Defects tracked + resolved
   
4. **GSC** (expanded)
   - Baselines formally established
   - Change requests logged + traced
   - Archive of baselines maintained
   
5. **MED** (expanded)
   - Metrics collected
   - Progress vs. plan measured
   - Issues reported (if variance >threshold)
   
6. Other areas: maintain Level G compliance

**Evidence to Collect:**
- Sprint plan / project schedule
- Requirements document (with version history)
- Risk register
- Code review records
- Defect tracking log
- Metrics dashboard (coverage, burndown, etc.)
- Change log (git history + acceptance)

**Certification Criteria:** Formal documentation for planning, execution, tracking + evidence of use.

---

## Level E (Parcialmente Definido)

**Goal:** Standardize organizational process (not just one project)

**Superset of F + new requirements:**

1. **GPR** (expanded)
   - Org-wide project process template created
   - Project adapts template to local context
   - Lessons learned captured (post-project)
   - Process improvement tracking
   
2. **GRE** (expanded)
   - Org-wide requirements process defined
   - Requirements templates standardized
   - Training on process for new team members
   
3. **OPP** — Orçamentação e Planejamento (Budgeting & Planning)
   - Org guidelines for planning + budgeting
   - Historical data used for estimation
   
4. **OPF** — Foco Organizacional (Organizational Focus)
   - Org strategy + process improvement strategy aligned
   - Quality policy documented org-wide
   
5. Other areas: maintain Level F compliance

**Evidence to Collect:**
- Org process document (e.g., `gestalt-kit/docs/process-template.md`)
- Project tailoring record (how local context adapted template)
- Post-project review notes
- Process improvement backlog
- Org quality policy
- Training materials + attendance

**Certification Criteria:** Process documented org-wide + evidence projects are using it + feedback loop for improvement.

---

## Level D (Definido)

**Goal:** Quantitatively manage processes

**Superset of E + new requirements:**

1. **GPR** (expanded)
   - Quantitative project goals (SLA, defect density, delivery date)
   - Process performance baselines established
   - Corrective actions triggered by metrics
   
2. **GRE** (expanded)
   - Requirements quantitative criteria (e.g., testability metrics)
   
3. **PCP** — Controle de Processo (Process Control)
   - Process performance monitored (vs. baseline)
   - Variances detected + corrected
   - Statistical control charts used
   
4. **AAL** — Análise de Causa (Causal Analysis)
   - Root cause analysis when process variance occurs
   - Preventive actions implemented
   
5. Other areas: maintain Level C compliance

**Evidence to Collect:**
- Quantitative process goals (written, tracked)
- Metrics baseline (historical data)
- Control charts (process performance)
- Root cause analysis records
- Corrective/preventive action log
- Metrics review meetings (data-driven decisions)

**Certification Criteria:** Quantitative goals + metrics collection + data-driven process adjustments.

---

## Levels C / B / A (Gerenciado Quantitativamente / Otimizado)

**Not active for Deviante yet.** Level D is the current ceiling.

**Summary:**
- **C**: Quantitative optimization within defined process
- **B**: Innovation systematically introduced + measured
- **A**: Continuous innovation + predictive capability

---

## Deviante — Current Assessment (Sample)

### UC1 (Login) — Status: F (Gerenciado)
- ✅ Tasks planned + tracked (sprint kickoff)
- ✅ Requirements documented (UC spec)
- ✅ Code review done (ActivitiesRepository PR)
- ✅ Tests written + passed
- ⚠️ Metrics collected? (partial — test coverage known, perf metrics not)
- **Path to E:** Document org process template; add performance SLA.

### UC4 (Event Log Upload) — Status: E (Parcialmente Definido)
- ✅ Tasks planned (quest doc)
- ✅ Requirements defined (A1/A2/B1/B2/B3 breakdown)
- ✅ Code review to happen (gate before ship)
- ✅ Org process adapted (follows gestalt-kit template)
- ⚠️ Metrics baseline? (design review checklist exists, not quantified)
- **Path to D:** Set SLA (upload latency <2s), defect density target, establish baseline.

### Top Nav (B3) — Status: F (Gerenciado)
- ✅ Tasks planned (part of UC4 quest)
- ✅ Design audit checklist created
- ⚠️ Requirements: Design spec doc missing
- ❌ Metrics: No responsive performance goals
- **Path to E:** Create design process doc; define accessibility criteria; add to org standard.

---

## MPS.BR Certification Checklist (for QA agent)

**Before certifying level X, verify:**

### For Level G ✓
- [ ] 7 process areas have evidence (task list, deliverables, etc.)
- [ ] Version control has commits
- [ ] Quality criteria identified (even if informal)
- [ ] Team knows what they're building

### For Level F ✓
- [ ] Project plan exists + is being followed
- [ ] Requirements documented + reviewed
- [ ] Defects tracked (if any found)
- [ ] Metrics collected (at least one: coverage, burndown, etc.)
- [ ] Risk register exists

### For Level E ✓
- [ ] Process template doc exists (org-wide, not just this project)
- [ ] This project documented how it tailored template
- [ ] Post-project review done (or scheduled)
- [ ] Quality policy org-wide
- [ ] New team members can follow process from docs

### For Level D ✓
- [ ] Quantitative goals set (SLA, defect density, cycle time)
- [ ] Baseline metrics established (historical data)
- [ ] Process performance monitored vs. baseline
- [ ] Variance triggers corrective action (documented)
- [ ] Root cause analysis for failures

---

## Related

- [qa.md](../../agents/qa.md) — QA agent (runs assessments)
- [maturity-assessor skill](../../skills/maturity-assessor/reference.md) — protocol
- [quality-gates skill](../../skills/quality-gates/reference.md) — enforcement at each level

# Historical Sprint Kickoff 23/07/2026 - Deviante UC3-UC4

> **Superseded:** this is a historical kickoff snapshot, not the active plan or
> current role model. Use
> [`sprint-plan-2026-07.md`](sprint-plan-2026-07.md). Alander is the sole
> developer/publisher; Eduardo Loures and Luiz Picolo are research partners and
> testers; agents are assistive automation roles.

**Status:** ✅ All preparation complete. Ready for developer + designer kickoff.

---

## What's Done (Preparation Phase — 21–22/07)

### 1. **Schema Gate (22/07 — APPLY FIRST)**
- ✅ SQL consolidated: `apply-schema-2026-07-22.sql`
- Tables: `deviante.activities`, `deviante.event_logs`, `deviante.operations`, `deviante.traces`, `deviante.drift_results`
- M:M junction: `process_activities` (activity catalog linking)
- **ACTION:** Apply via Supabase Web UI, DataGrip, or psql (see instructions)

### 2. **Backend Infrastructure (Deviante API)**
- ✅ `ActivitiesRepository` + `ActivitiesTable` (UC3 catalog CRUD)
- ✅ `OperationsRepository` + `OperationsTable` (UC5/UC6 operation mapping)
- ✅ API routes: `/api/activities`, `/api/operations/{id}/map`, `/api/operations/{id}/unmap`
- ✅ Role-based access: owner/mentor see all processes, managers see own only
  - Owner: `design@alander.io`
  - Mentor: `pafileiro@gmail.com` (Luiz Picolo)
- ✅ Demo data seed: 4 ready-to-demo processes (manufacturing, retail, finance, credit)
- **STATUS:** Deployed to `aland3r/deviante-api` (commit `fab3250`)

### 3. **Frontend Components (Deviante Web)**
- ✅ `OperationsList.jsx` — UC5/UC6: list operations, map to activities
- ✅ `OperationMappingModal.jsx` — modal for activity selection
- ✅ `ActivitiesManager.jsx` — UC3: manage shared catalog
- ✅ Updated `README.md` with Radix UI + shadcn/ui + Tailwind v4 stack
- **STATUS:** Deployed to `aland3r/deviante-web` (commit `97bd6ed`)

### 4. **Tools & Monitoring**
- ✅ `/audit-merges` command — polyrepo merge conflict detection
- ✅ `/gamifier-sync` command — quest log consistency audit
- ✅ `/sync-kit` policy — auto-sync gestalt-kit → Supabase portfolio.kit_docs
- **STATUS:** All committed to `gestalt-hub`

### 5. **Quests & Planning**
- ✅ UC4 quest breakdown: backend (A1/A2), frontend (B1/B2), design (B3)
- ✅ Dependency map: A1 → A2 → {B1, B2} → integration → ship
- ✅ Blocker: Top nav fix (B3) must complete before Vercel deployment
- **STATUS:** Quest document ready (gestalt-kit/plans/UC4-EVENT-LOG-UPLOAD-QUEST.md)

---

## What's Next (Active Sprint — 23–24/07)

### Timeline

| Day | Phase | Owner | Tasks | Blocker |
|-----|-------|-------|-------|---------|
| **23/07 AM** | **Gate + Backend** | Schema reviewer + Dev | Apply schema SQL (DataGrip); A1 (EventLogsRepository); A2 (upload handler) | Schema must apply cleanly |
| **23/07 PM** | **Frontend + Design** | Dev + Designer | B1 (OperationsList tab); B2 (EventLogUploadForm); B3 (top nav fix) | A2 completion blocks B1/B2 |
| **23/07 EOD** | **Integration Test** | Dev + QA | Upload → operations list → confirm → save flow | B3 not ready = defer ship |
| **24/07 AM** | **Blocker Fixes** | Dev + Designer | Fix any B3 responsive issues; A2 edge cases; staging test | B3 must green-light |
| **24/07 PM** | **Ship to Vercel** | Polyrepo-shipper | Deploy deviante/api + deviante/web | All tests pass + designer approves |

### Critical Path

```
Schema Apply (AM 23/07)
    ↓
A1 EventLogsRepository (→ 11am)
    ↓
A2 Upload Handler (→ 2pm)
    ↓
B1/B2 Frontend (2pm → 6pm)
    ↓
Integration Test (6pm → 8pm)
    ↓
B3 Designer Approval (done in parallel, needs by EOD)
    ↓
Ship to Vercel (24/07 PM)
```

### Parallel Work

- **B3 (Top Nav Fix):** Can start immediately after design kickoff (doesn't wait for A2)
- **EventLogUploadForm (B2):** Can start after A2 routes are defined (can mock endpoints)
- **OperationsList integration (B1):** Blocked on A2, but UI already exists

---

## Developer Checklist (23/07)

### Backend (A1 + A2)

- [ ] **A1: EventLogsRepository**
  - [ ] Create `db/EventLogsTable.kt` (if not auto-derived from migration)
  - [ ] Create `repository/EventLogsRepository.kt` with:
    - `listForProcess(processId)`
    - `create(processId, fileName, format)` → returns EventLogRecord
    - `updateParseStatus(eventLogId, status)` → pending/parsing/parsed/failed
    - `delete(eventLogId, processId)` → cascade deletes operations + traces
  - [ ] Create `model/EventLogRecord.kt` + `dto/EventLogDto.kt`
  - [ ] Add unit tests (mock Supabase, verify CRUD)

- [ ] **A2: Multipart Upload Handler**
  - [ ] Route: `POST /api/processes/{id}/event-logs`
  - [ ] Accept multipart file (CSV or XES)
  - [ ] Validate: file size <100MB, extension ∈ {.csv, .xes}
  - [ ] Auth check: manager owns process
  - [ ] Save to DB: `deviante.event_logs` (parseStatus = pending)
  - [ ] Extract operations: stub or call PM4Py
    - [ ] Return OperationRecord[] with rawLabel, occurrenceCount
  - [ ] Response: EventLogRecord + operations list
  - [ ] Error handling: file validation, network, parse timeouts

- [ ] **Integration Test**
  - [ ] Create test file (sample CSV with operations)
  - [ ] Call POST endpoint
  - [ ] Verify event log created
  - [ ] Verify operations extracted
  - [ ] Verify database state

### Frontend (B1 + B2)

- [ ] **B1: OperationsList Integration**
  - [ ] Add tab/section to `ProcessDetailPage.jsx`
  - [ ] Import `<OperationsList eventLogId={eventLogId} />`
  - [ ] Pass props correctly
  - [ ] Conditional render (show if event log exists)

- [ ] **B2: EventLogUploadForm (new component)**
  - [ ] Create `EventLogUploadForm.jsx`
  - [ ] File input (accept .csv, .xes)
  - [ ] Upload button → POST /api/processes/{id}/event-logs
  - [ ] Progress bar (if chunked upload, else fake progress)
  - [ ] Error handling (validation, network)
  - [ ] Success state: show extracted operations table
  - [ ] Confirm button: trigger save (or just immediate on success?)
  - [ ] Integrate into ProcessDetailPage

### Designer (B3)

- [ ] **Top Nav Fix (AppLayout.jsx)**
  - [ ] Audit `src/components/layout/AppLayout.jsx`
  - [ ] Check: logo sizing (mobile ≤40px), nav spacing, user menu alignment
  - [ ] Convert CSS to Tailwind + responsive classes
  - [ ] Dark/red theme: verify color classes applied
  - [ ] Test on mobile (375px preset) — no horizontal scroll
  - [ ] Test on tablet (768px preset) — spacing OK
  - [ ] Test on desktop (1280px preset) — theme colors correct
  - [ ] Approve for ship: "green light" or "needs fix by 24/07 AM"

---

## Designer Checklist (23/07)

- [ ] Audit `AppLayout.jsx` header structure
- [ ] Test responsive breakpoints (375/768/1280px)
- [ ] Verify Tailwind classes are applied
- [ ] Verify dark/red theme colors (check Figma export vs live)
- [ ] Fix issues: logo overflow, nav cramping, menu inaccessible on mobile
- [ ] Staging deployment test on Vercel preview
- [ ] Approve/flag for ship

---

## Quality Gate (EOD 23/07)

- [ ] All A1/A2 unit tests pass
- [ ] B1/B2 integration tests pass (upload → operations list)
- [ ] B3 designer approves top nav (or documents fixes for 24/07 AM)
- [ ] No console errors on staging
- [ ] Mobile/tablet/desktop all verified
- [ ] Code review sign-off (optional, if time permits)

---

## If Slipping

- **A2 not done by 2pm?** Defer B1/B2 to 24/07 AM, compress testing to 2 hours
- **B3 not done by 6pm?** Defer deployment to 24/07 PM; document blocker for polyrepo-shipper
- **A3 (FastAPI) not ready?** Stub with empty operations list; note as tech debt

**Escalate blockers to product-manager by noon 23/07.**

---

## Ship Conditions (24/07 PM)

✅ All developer tasks done
✅ All tests pass
✅ Designer approves top nav
✅ No new console errors
✅ Staging preview looks good on mobile
✅ Vercel deployment succeeds

**Then:** Polyrepo-shipper publishes `deviante/api` + `deviante/web` to production.

---

## Contact Matrix

| Role | Owner | Slack/Contact | Blocker Escalation |
|------|-------|---------------|-------------------|
| Backend Dev (A1/A2) | [Name] | [Handle] | Product Manager |
| Frontend Dev (B1/B2) | [Name] | [Handle] | Product Manager |
| UI Designer (B3) | [Name] | [Handle] | Product Manager |
| Product Manager | Alander (Owner) | design@alander.io | N/A |
| Shipper (24/07 PM) | Polyrepo-shipper agent | [Automation] | N/A |

---

## Definitions of Done

**UC4 Happy Path (Developer PoV):**
1. User uploads CSV/XES file
2. System extracts operation labels (raw text)
3. UI shows "Confirm? 42 operations extracted"
4. User clicks Confirm
5. Operations saved to DB (mapping_status = "unmapped")
6. Operations tab shows list, ready for mapping to activities (UC5/UC6)

**Top Nav (Designer PoV):**
1. No horizontal scroll on mobile (375px)
2. Logo visible + correctly sized
3. Nav links readable (not cramped)
4. User menu accessible (dropdown doesn't overflow)
5. Dark/red theme applied (colors, shadows, borders match Figma)
6. Responsive on tablet (768px) + desktop (1280px)

**Deployment:**
1. Vercel staging preview works
2. No console errors (dev tools, mobile)
3. Mobile manual test: upload → operations confirm flow
4. Designer green-light on top nav
5. Polyrepo-shipper deploys to production

---

## Links & Resources

- **Schema:** `data/schema/deviante/apply-schema-2026-07-22.sql`
- **Demo Seed:** `data/schema/deviante/seed-demo-processes.sql`
- **Quest:** `gestalt-kit/plans/UC4-EVENT-LOG-UPLOAD-QUEST.md`
- **Backend Repo:** `aland3r/deviante-api` (main branch)
- **Frontend Repo:** `aland3r/deviante-web` (main branch)
- **Staging:** https://deviante-web-staging.vercel.app (or preview link)
- **Production:** https://deviante.alander.io

---

**🚀 Ready for 23/07 AM kickoff. Developers start A1/A2; designers start B3 audit. Report blockers by noon.**

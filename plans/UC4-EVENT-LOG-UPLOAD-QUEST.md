# UC4: Event Log Upload Quest (Design + Dev)

**Sprint:** 23/07/2026 (happy path focus)

**Owner:** Gamifier + UI Designer

**Scope:** Complete UC4 happy path (upload event log CSV/XES, extract operations, confirm before save) + fix top nav UI inconsistency

---

## Quest Breakdown

### Part A: Backend (EventLogsRepository + Upload Handler) — Dev Focus

**Task A1:** EventLogsRepository (Kotlin)
- CRUD for `deviante.event_logs` table
- List event logs for a process
- Create event log (track filename, format, parse_status)
- Update parse_status as async parsing completes
- Delete event log (cascade to operations, traces)

**Task A2:** Multipart Upload Handler (Ktor)
- Route: `POST /api/processes/{id}/event-logs` (auth required)
- Accept file upload (CSV or XES)
- Validate format (extension check)
- Save to temp storage or return immediately
- Extract raw operation labels (async via FastAPI?)
- Create operations rows with mapping_status = "unmapped"
- Return list of extracted operations for UI confirmation

**Task A3:** Connect to FastAPI (if event log parsing needed)
- Call external process-mining service for parsing
- Or use PM4Py wrapper (if available)
- Return structured traces + operations

**Status:** Tasks A1-A2 are blockers for UI. A3 can be async.

---

### Part B: Frontend (OperationsList integration + Top Nav fix) — Design + Dev Focus

**Task B1:** Add Operations Tab to ProcessDetailPage
- Import `<OperationsList processId={processId} eventLogId={eventLogId} />`
- Show after file upload succeeds
- Link to ActivitiesManager for catalog reference

**Task B2:** Event Log Upload UI Component
- Form: file picker (accept .csv, .xes)
- Upload button → multipart POST
- Progress bar during upload
- Show extracted operations table (A2 response)
- Confirm button → save to DB
- Error handling (file validation, network, parse failures)

**Task B3: TOP NAV VISUAL FIX** (Separate issue, **blocking this quest**)
- Current issue: Top nav shows inconsistent spacing/alignment on mobile
- State: AppLayout shell → header with logo, nav links, user menu
- Problem: Responsive design breaks on <768px (tablet)
- Task:
  - Audit current AppLayout header (deviante/web/src/components/layout/AppLayout.jsx)
  - Fix spacing, align user menu to right
  - Test on mobile (375px preset)
  - Ensure logo/nav don't overflow
  - Ensure dark/red theme (Tailwind + shadcn) is applied consistently
  - Verify on Vercel staging before ship

**Status:** B1-B2 ready once A1-A2 done. B3 **must complete before ship**.

---

## Design Checklist (UI Designer)

- [ ] Review AppLayout header (top nav)
  - [ ] Logo sizing (should not exceed 40px height on mobile)
  - [ ] Nav links spacing (check mobile breakpoint)
  - [ ] User menu button (dropdown alignment on mobile)
  - [ ] Dark/red theme: verify Tailwind classes are correct
  - [ ] No horizontal scroll on mobile
  
- [ ] Review EventLogUploadModal (new component)
  - [ ] File input styling (native or custom?)
  - [ ] Progress bar UX (smooth, clear percentage)
  - [ ] Error messages (readable, actionable)
  - [ ] Confirm button placement (mobile-friendly)
  
- [ ] Review OperationsList + OperationMappingModal (existing, verify alignment)
  - [ ] Table responsive on mobile (horizontal scroll OK, not ideal)
  - [ ] Modal dialog on mobile (should fill >80% of screen width)
  - [ ] Badge styling (mapping status colors clear?)

---

## Developer Checklist

- [ ] A1: EventLogsRepository.kt created
  - [ ] listForProcess(processId)
  - [ ] create(processId, fileName, format)
  - [ ] updateParseStatus(eventLogId, status)
  - [ ] delete(eventLogId, processId)
  
- [ ] A2: Upload endpoint created
  - [ ] Route: POST /api/processes/{id}/event-logs
  - [ ] Auth check (manager owns process)
  - [ ] Multipart parsing (accept .csv, .xes)
  - [ ] Validation (file size <100MB, extension valid)
  - [ ] Save to DB (deviante.event_logs)
  - [ ] Extract operations (stub or call PM4Py)
  - [ ] Return operations list
  
- [ ] B1: OperationsList integrated
  - [ ] Add tab/section to ProcessDetailPage
  - [ ] Pass eventLogId prop
  - [ ] Show upload UI or list
  
- [ ] B2: EventLogUploadForm component created
  - [ ] File picker input
  - [ ] Upload handler (call POST /api/processes/{id}/event-logs)
  - [ ] Progress/loading state
  - [ ] Error handling
  - [ ] Success → show operations confirmation
  
- [ ] B3: Top nav responsive fix
  - [ ] Audit AppLayout.jsx
  - [ ] Fix mobile spacing/alignment
  - [ ] Verify theme colors (dark/red)
  - [ ] Test on 375px + 768px + 1280px viewports

---

## Definition of Done

✅ UC4 Happy Path:
1. User uploads event log (CSV/XES)
2. System extracts operation labels
3. UI shows "Confirm? N operations extracted"
4. User confirms → operations saved
5. Operations tab shows list (ready to map to activities)

✅ Top Nav:
1. No horizontal scroll on mobile
2. Logo visible + properly sized
3. Nav links readable (not cramped)
4. User menu accessible on mobile
5. Dark/red theme applied correctly

✅ Deployment:
1. No console errors
2. Mobile preview tested (375px)
3. Tablet preview tested (768px)
4. Desktop preview tested (1280px)
5. Vercel staging deployment working

---

## Timeline

| Day | Task | Owner |
|-----|------|-------|
| 23/07 AM | A1 + A2 backend | Dev |
| 23/07 PM | B1 + B2 frontend (blocking on A1/A2) | Dev + Designer |
| 23/07 PM | B3 top nav fix (parallel) | Designer |
| 23/07 EOD | Integration test (upload → operations list) | Dev + QA |
| 24/07 AM | Fix blockers (if any) | Dev |
| 24/07 PM | **Ship to Vercel** | Shipper |

---

## Related Quests

- **UC3-1a:** Request access (done)
- **UC3-1b:** Admin pending queue (done)
- **UC3 happy path:** CRUD activities (23/07)
- **UC4 happy path:** Upload event log + extract operations (THIS QUEST, 23/07)
- **UC5/UC6 happy path:** Map operations to activities (24/07)

---

## Notes for Gamifier

- Status: `in_progress` (after approval)
- Linked UC: `ABP-DV-UC4` (Event Log Upload)
- Progress: 0/3 tasks (A1, B1+B2, B3)
- Blocker on top nav (B3): if not resolved, cannot ship
- Estimated: 1 day (23/07)
- Risk: PM4Py integration (A3) — can be stubbed for demo

---

## Rollback / Abort Criteria

- If A1 implementation takes >4 hours (stub instead of full CRUD)
- If file upload multipart parsing fails repeatedly (use simpler approach)
- If top nav fix requires major refactor (defer to 24/07)

In all cases, report to product-manager before aborting.

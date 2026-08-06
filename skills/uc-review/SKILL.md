---
name: uc-review
description: >
  OOUX early clarification + UX text review for one UC before implementation.
  Calls oouxer (OOUX objects/roles/CTAs), then ux-writer (UC text quality).
  Use before coding a UC to align objects and validate descriptions.
disable-model-invocation: true
argument-hint: [abp-id]
---

Run the **UC review gauntlet** for `$0` (e.g. `ABP-DV-UC3`): OOUX early
clarification, then UX text review.

1. Read [reference.md](reference.md) for the workflow.
2. **Phase 1: OOUX Early Clarification** — Call `oouxer` to workshop
   Objects, Roles, CTAs, Attributes for this UC against the Deviante ORCA Hub
   (Notion). Owner is stakeholder.
3. **Phase 2: UX Text Review** — Call `ux-writer` to review UC descriptions
   (why/what/bounds) and steps against the OOUX clarifications + writing
   quality bar.
4. Aggregate outcomes: object cards, text revisions proposed.
5. Report to owner for approval before implementation (`/uc-gate` signals
   readiness).

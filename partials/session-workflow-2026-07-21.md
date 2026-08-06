# Session Work: 2026-07-21 Eve — Login Perf Fix, OAuth Investigation, Workflow Policy

**Date:** 2026-07-21 23:30+  
**Work Status:** Complete (commit `0d67620` merged to master)  
**Related Issues:** Deviante login performance regression, Supabase OAuth redirect URI misconfiguration

---

## Summary

This session addressed a critical login performance issue in Deviante caused by malformed cookie reads, stabilized the commit workflow policy, and clarified the unified Supabase architecture for Portfolio + Deviante.

### Commit: `0d67620` — Fix Deviante login performance

**Changed files:**
- `ui/auth/cookie-storage.js` — Disable cookie storage reads (return null, force localStorage fallback)
- `index.html` — Add inline cleanup script for malformed cookies (runs before module load)
- `gestalt-kit/skills/uc-validate/SKILL.md` — New skill definition
- `gestalt-kit/skills/uc-validate/reference.md` — Validation reference documentation

**Why:** Cookie storage reads were throwing URI decode errors on every Supabase auto-refresh (every ~5 seconds), freezing login UX. This was a regression, likely from stale Vite cache during bundling.

**Root cause:** Supabase client in bundle retained old version; disabling reads forces fallback to localStorage (which works correctly). Hard browser refresh clears this; production deploy will resolve fully.

---

## Team Memory Documentation

Three key decisions documented in session memory (stored in `.claude/projects/C--gestalt/memory/`):

### 1. Commit Workflow: Polyrepo-Shipper Only

**Policy:** All code commits must go through the `polyrepo-shipper` agent, never direct `git commit` CLI.

**Why:**
- Centralized control and auditability
- Includes chat/transcript context for traceability
- Consistent workflows across sessions and agents

**How to apply:**
1. Code changes complete + tested
2. Use Agent tool → launch `polyrepo-shipper`
3. Provide: file list, commit message (WHY + WHAT), chat excerpts for context
4. Include overlapping session context for any code touched
5. Never use `git commit`/`git push` directly
6. Report PR URL when done

**This memo:** Ensures accountability and context preservation for all work going forward.

### 2. Supabase: Unified Project Architecture

**Fact:** Single Supabase project `ydjtrcjxhtmygytmebrk` hosts both products:
- **Portfolio (IO)** — `portfolio.*` schema, tables
- **Deviante (DV)** — `deviante.*` schema, tables
- Shared Auth → Google OAuth (single config for both)

**Key URLs:**
- Portfolio: https://portfolio.alander.io
- Deviante: https://deviante.alander.io
- Supabase API: https://ydjtrcjxhtmygytmebrk.supabase.co

**Implication for auth fixes:** When debugging OAuth issues (like redirect URI misconfiguration), remember ONE project, TWO products. Both products' redirect URIs must be registered in a single Google OAuth app config.

### 3. Commit Message Style

**Standard:** Subject line starts with capital letter, followed by description.

**Example (this session):**  
```
Fix Deviante login performance: disable malformed cookie reads

Cookie storage reads were throwing URI decode errors on every Supabase
auto-refresh (every few seconds), freezing login. Disabled cookie reading
entirely in cookie-storage.js (returns null), forcing fallback to
localStorage...
```

---

## Investigation Notes: OAuth Redirect URI Checklist

During OAuth debugging in production, confirmed the following checklist items for Supabase Google auth setup:

**Supabase Dashboard → Authentication → Providers → Google:**
- [ ] `GOOGLE_CLIENT_ID` set
- [ ] `GOOGLE_CLIENT_SECRET` set
- [ ] Authorized JavaScript origins include:
  - `https://portfolio.alander.io`
  - `https://deviante.alander.io`
  - `https://ydjtrcjxhtmygytmebrk.supabase.co` (Supabase URL)
- [ ] Authorized redirect URIs include:
  - `https://portfolio.alander.io/auth/callback`
  - `https://deviante.alander.io/auth/callback`
  - `https://ydjtrcjxhtmygytmebrk.supabase.co/auth/v1/callback` (Supabase callback)
- [ ] Verify in Google Cloud Console → APIs & Services → OAuth 2.0 Credentials that URIs match exactly (no trailing slashes, protocol must be https)

---

## Context for Next Session

- **Schema work (Deviante)** still pending (see sprint plan 22/07): `activities.sql`, `event_logs.sql`, `operations.sql`, `traces.sql` are rascunhados (drafted but not applied to Supabase live).
- **UC esteira:** UC1–UC2 marked `spec_confirmed`; UC3–UC13 still `unchecked` (see `gestalt-kit/partials/uc-esteira.md`).
- **Portfolio infra:** RLS policies for `tracks`/`quests` still missing (see sprint plan "Status de infra" section).
- **Next blocker (22/07):** Database gate — `architect` + `database-integrations` review + apply schema, `/uc-gate` UC3/UC4.

---

## Files Modified This Session

| File | Change | Type |
|------|--------|------|
| `ui/auth/cookie-storage.js` | Disable reads, force localStorage | Fix |
| `index.html` | Cleanup script for malformed cookies | Fix |
| `gestalt-kit/skills/uc-validate/SKILL.md` | New skill definition | Addition |
| `gestalt-kit/skills/uc-validate/reference.md` | Validation reference | Addition |
| `gestalt-kit/partials/session-workflow-2026-07-21.md` | This file — session summary | Documentation |

---

## Related Commits in Branch History

- `a9b53ff` Add QA agent, MPSBR certifier skill, alternative flows gate rule
- `2113fb4` Sprint kickoff brief: UC3–UC4 preparation complete, ready for 23/07 AM dev start
- `90ad7e5` Create UC4 event log upload quest: backend (EventLogsRepository) + frontend UI + top nav fix

Commit `0d67620` is the capstone for this session's work; see `git show 0d67620 --stat` for full diff.

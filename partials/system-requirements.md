<!--
PARTIAL: cross-cutting system requirements — non-negotiable constraints that
apply across every product and UC, not product-specific or UC-specific.
Link from: kit-navigation (cold start), architecture, uc-esteira,
write-use-case, deviante-backend. Do not copy-paste — link this file.
-->

# System requirements

Hard, cross-cutting rules. Every agent treats these as constraints, not
suggestions. Detailed mechanics live in the doc each rule points to — this
file is the index of "things that are always true," not where they're
implemented.

## SR1 — Language

**Code and gestalt-kit knowledge docs (`agents/`, `skills/`, `partials/`,
`docs/`) are written in English**, going forward. This is already the
practice for `docs/architecture.md`, most of `partials/`, and UC content in
`portfolio.use_cases` — new material should stay consistent with it.

**Not covered by this rule:**
- `gestalt-kit/plans/sprint-plan-*.md` — the owner's own day-by-day working
  notes, not a kit knowledge artifact. Stays PT-BR.
- Chat conversation itself — agents match the owner's language (PT/EN), see
  `maestro`'s Voice section.
- Existing PT-BR content elsewhere is **not** retroactively translated as
  its own task — fix opportunistically when a file is touched for another
  reason.

## SR2 — Deviante backend stack

**Deviante's persistence and business logic are Kotlin (OOP, Exposed) end to
end.** `deviante/web` (TS/React) only calls the Ktor API — it never
reimplements persistence or business rules in JS/TS. **Portfolio is the
sole exception** (may use `supabase-js` directly). This is a standing
assurance `deviante-backend` re-checks on every task, not a one-time setup
step. Full detail + the agent-level enforcement duty:
[architecture.md § Endorsed patterns](../docs/architecture.md) (row
"Deviante stack boundary") and [deviante-backend.md](../agents/deviante-backend.md).

**One narrow exception:** process mining computation (CSV/XES → visual
trace, UC4 onward) runs in a separate Python + FastAPI service using
PM4Py — no JVM equivalent exists. Kotlin still calls it and persists the
result; see [architecture.md § Process mining
exception](../docs/architecture.md). This is one named exception, not
precedent for JS/TS persistence elsewhere.

## Adding a new requirement

Append `SR{n}` here: the rule in one short paragraph, then which existing
doc (if any) owns the detailed enforcement. Do not duplicate the detail
here.

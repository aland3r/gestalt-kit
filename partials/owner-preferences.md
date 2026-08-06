<!--
This is a PARTIAL: a shared fragment meant to be linked from multiple skill
files, never copy-pasted into them. If you catch yourself pasting this text
into a new skill, link here instead — that's the whole point of a partial.

Referenced from:
- gestalt-kit/skills/gestalt-context/reference.md
- .cursor/skills/gestalt-context/SKILL.md
- .claude/skills/gestalt-context/SKILL.md
- gestalt-kit/skills/gestalt-context/SKILL.md
-->

# Owner preferences - no clutter

**No clutter.** The owner rejects UI and code clutter - do not add disabled placeholders, "coming soon" controls, extra providers, or speculative options "for later." Ship only what v1.0 needs; add alternatives when they are actually implemented.

Also enforce **[active-scope.md](active-scope.md)** (IO+DV only; never Flashbrix) and **[repo-consistency.md](../repo-consistency.md)** / **[prefer-existing-files.md](../prefer-existing-files.md)**.

## Working rhythm

- Review a UC while implementing it, not in a long specification-only phase.
- Ask only the concrete questions that block the current UC. Do not interview
  the owner about later UCs before the current happy path works.
- Keep the UC gate: load the live Supabase card, persist owner edits, read them
  back, and get an explicit implementation confirmation.
- Deliver one vertical slice at a time on the real deployed product. The
  owner's acceptance phrase is **"amei, next"**.
- Functionality comes first. Keep touched surfaces visually consistent with
  the latest Figma Make reference, but do not delay a working flow for a broad
  redesign.
- Deviante's rotating `figma-make/` drop is temporary implementation input:
  inspect the newest ZIP/source, reproduce the approved surface in the real
  product with high fidelity, then allow the drop to be replaced or deleted.
  Do not treat generated prototype code as a production dependency.
- Figma owns the primary visual solution. It does not own product language:
  `oouxer` guards object and CTA vocabulary, `ux-writer` owns shipped interface
  strings, and `ux-engineer` owns hierarchy and fit. `researcher` may provide
  user evidence and comprehension insights; `ui-designer` reconciles the
  approved language into the Figma-led visual composition.
- Deviante UI copy is PT-BR. Code and knowledge documentation are English.
- Production is the acceptance environment for Deviante. A local-only demo is
  not a finished slice.

## Product stewardship

- Alander is Deviante's sole product owner, developer, publisher, and final
  decision-maker. Agents assist his work; they are not human team members or
  independent owners of deliverables.
- Eduardo Loures is Alander's PIBITI advisor, a PUCPR Mechatronics professor,
  research partner, and qualified product tester.
- Luiz Picolo is a master's researcher working at Bosch, a research partner,
  source of relevant ADWIN/dataset work, and a qualified product tester.
- Eduardo and Luiz do not publish Deviante and should not be assigned product
  delivery tasks in plans. Their access exists for research collaboration,
  technical feedback, and testing.
- Deviante runtime roles are `owner` for Alander and `mentor` for Eduardo and
  Luiz. Mentor is an access/testing role, not product ownership.
- Deviante is a live PIBITI product with commercial and intellectual-property
  ambition. Treat patenting and market sale as strategic goals, not as an
  already established legal status.
- Repository cohesion is a product requirement: when code, UCs, plans,
  architecture, agents, skills, or commands disagree, surface and resolve the
  drift in the owning source of truth instead of adding another parallel note.

Examples:

- **Accounts:** seed SQL + owner distributes credentials — no self-service register unless explicitly scoped ([seed-accounts.md](../seed-accounts.md))
- Auth: Google OAuth only in Deviante v1.0 — no disabled GitHub/Microsoft buttons
- Features: omit menu items, tabs, and variants that are not wired up yet
- Docs/skills: keep lists minimal; avoid duplicating the same rule in many files — link a partial instead

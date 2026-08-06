<!--
This is a PARTIAL: link from skills/agents, never copy-paste.
Referenced from:
- gestalt-kit/agents/ooux.md
- design-system/README.md
-->

# OOUX vocabulary — ORCA names win

**Same nouns everywhere (per product).** Screens, Figma frames, React copy,
and code identifiers that name domain things must match that product's
Notion **ORCA Hub** (Objects, Roles, CTAs, Attributes). The Hub is the
source of truth for *what we call things* in the product.

**Do not confuse with the method SoT.** How to run ORCA is learned from the
Notion DB **OOUX Masterclass** (video subtitles / lesson notes):
https://app.notion.com/p/3065fc7249408094925cce4447bf5862
Masterclass examples are curriculum — not this product's object list.

Schema and quests follow after the Hub name is decided; they do not invent
competing labels. The `ooux` agent learns and executes ORCA; it does not
replace the Hub with chat vocabulary.

Examples:

- Before a new button or menu item: find (or workshop) the **CTA** on the
  right **Object** for the acting **Role**
- Do not rename `Manager` to "User" in the UI because the table is
  `managers` — align UI to ORCA, or change ORCA explicitly with the owner
- If ORCA and the live UI disagree, that is **drift** — fix the replica or
  update the Hub on purpose; never paper over with a third name
- Deviante Hub: ORCA Hub 2.2
  (page `27b5fc72-4940-8229-a5c6-01c46e28946f`)

Related: agent `ooux`; `design-system/README.md` (ORCA → ADS → code).

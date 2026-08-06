# Alander — Owner Operator (IO)
Product: IO
ORCA Role: Owner (portfolio governance)

## Context

Alander maintains the Gestalt portfolio hub (`alander.io`) as owner: use cases,
kit docs, publications, and site copy. He edits content directly in Supabase
(DataGrip) or on `/kit` and `/cases` to save AI tokens, then runs **depara**
when agents must catch up.

## Goals / jobs

- See at a glance whether a UC or kit doc is **live** (public), **private**
  (owner-only), or **offline** (draft) — without opening the editor.
- Navigate `/cases` and `/kit` as owner and trust the **publication beacon**
  (green / blue / dim flame) matches what anonymous visitors actually get.
- Change visibility in DataGrip or UI without re-explaining state to every agent.

## Frustrations

- Text labels (`draft`, `owner`) that do not read as “on air vs private”.
- Drift between git kit and live DB after manual edits — needs depara, not guesswork.
- Beacons that pulse too loud or clash with the copper logo flame.

## Scenario seed

> I open Cases as owner, scan the green dots on Deviante UCs that are public,
> spot a blue one I forgot to publish, and fix visibility in DataGrip — no chat
> with an agent until I ask for depara.

## Evaluation checklist (publication beacon)

When reviewing the site as this persona:

1. **Legend** visible on `/cases` and `/kit` when logged in as owner.
2. **Green pulse** on rows with `visibility=public` and `status` ready/shipped.
3. **Blue pulse** on owner/member or ready-but-not-public items.
4. **Dim dot** on draft/deprecated — reads as offline, not broken.
5. Beacon size does not collide with UC id, edit buttons, or kit card title.
6. `aria-label` / tooltip matches locale (en/pt/de).
7. Anonymous `/cases` view: only public UCs visible; beacons (if shown) are green.
8. **Autolayout** — filter chips have visible 8px gaps; detail pane framed when
   empty (`partials/autolayout-ux.md`).

## Appears in

- Portfolio `/cases`, `/kit` — publication beacon UX
- `partials/kit-depara.md` — owner DB-first workflow
- truth-keeper `/kit-depara` after manual Supabase edits

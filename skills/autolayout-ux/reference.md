# autolayout-ux — procedure

Read **[partials/autolayout-ux.md](../../partials/autolayout-ux.md)** first.

## When implementing layout

1. Name the **organism** (e.g. kit filter bar, split pane).
2. Pick container pattern: **stack** | **cluster** | **split** | **grid**.
3. Assign gaps from `--cases-space-8|16|24|32` only.
4. Atoms inside molecules get **16px** internal padding unless chip-tight (8).
5. Every pane that can be empty gets a **framed empty state** (border + min-height).
6. `min-width: 0` on grid/flex children that hold truncating text.

## When reviewing (checklist)

- [ ] Sibling controls have ≥ 8px gap (not shared side borders only)
- [ ] Page uses vertical stack 24px between major sections
- [ ] Split view: both columns bounded; no 60% void with one line of text
- [ ] Cards: grid gap 8; no overlapping beacons/titles
- [ ] Persona can scan status + title without reading body (Owner-Operator)
- [ ] Mobile: split stacks; touch targets ≥ 40px where interactive

## CSS class vocabulary (Portfolio /kit)

| Class | Role |
|-------|------|
| `.kit-stack--gap-16` | Vertical section spacing |
| `.kit-cluster--gap-8` | Filter chips, tag rows |
| `.kit-split` | List + detail grid |
| `.kit-panel__detail--empty` | Framed pick-a-doc state |

Extend the same vocabulary on other routes (`cases-*` prefixes) — do not invent
parallel spacing systems.

## Handoff

| Finding | Agent |
|---------|--------|
| Spacing, cluster, empty pane | `ui-designer` + this skill |
| Expand/flicker | `interaction-designer` |
| Persona cannot complete task | `persona-crafter` update checklist |
| ORCA labels | `ooux` |

# Odair — Color-Vision Accessibility (cross-product)

Product: DV (origin), applies cross-product — see [[ux-researcher]]
ORCA Role: workshop pending — cross-cutting accessibility constraint, not a
product actor

## Context

Odair is a maintenance manager on the shop floor with **deuteranopia**: he does
not perceive green. Greens read to him as dull yellows, browns, or grays, and
he cannot tell green from red or green from a neutral. He reads Deviante on a
good monitor in a bright plant, and he is exactly the sibling of [[Miguel]]:
not a persona for one feature, but the concrete "why" behind the rule **meaning
must never be carried by hue alone**.

For Odair, any signal encoded only as green vs. red — or green as the sole
identity of a thing — is invisible or ambiguous. He compensates with position,
labels, and brightness, but only when the design gives him those to lean on.

## Goals / jobs

- Read a machine illustration, a status, or a drift marker and know what it
  means without having to distinguish green.
- Tell "monitoring / healthy" from "drift / alarm" by something other than a
  green-vs-red color pair — a shape, a label, a position, a brightness step.
- Trust that an accent color is never the *only* thing separating two states.

## Frustrations

- **Green as the only signal**: a health line, an "OK" badge, or a
  monitoring-identity icon rendered in green with no shape or label backup.
- **Red/green pairs**: drift vs. baseline (or alarm vs. healthy) distinguished
  only by red vs. green — the two states collapse into one for him.
- **Low-luminance line art**: thin dark strokes on a dark panel, where the only
  thing rescuing legibility for others is a hue difference he cannot see (the
  original dark-mode machine illustration — near-black on near-black).
- Charts where series are told apart by hue alone (green vs. teal vs. olive).

## Scenario seed

> A monitoring card shows a rising "health" trend in green with a small green
> tower icon as its identity. To me the line and the icon are a muddy gray on a
> dark card — I can't tell this card is a monitoring instance, and I can't tell
> the trend is the *good* one, until I read every label.

## Evaluation checklist (color-vision accessibility)

When reviewing a surface as this persona:

1. **No meaning by hue alone** — every state, category, or identity encoded in
   color also carries a second channel: shape, icon, label, position, or a
   clear **luminance** step. Remove color mentally — is it still readable?
2. **No bare red/green oppositions** — healthy vs. drift, pass vs. fail, on vs.
   off must differ by more than red vs. green (add ✓/!, fill vs. outline,
   up/down, text).
3. **Illustrations read by luminance, not hue** — schematic art (e.g.
   `MachineIllustration`) keeps a real light/dark contrast between body, panel,
   and strokes, so it survives a grayscale pass (`--illo-*` tokens exist for
   exactly this). No green used as an illustration accent.
4. **Categorical charts** — series are separable in grayscale (vary lightness /
   add direct labels or markers), not only by hue.
5. **Green is never a lone identity** — if a feature (e.g. Monitoramento) uses
   green as its brand accent, its identity is also carried by an icon shape and
   a text label, never the color by itself.
6. **Verify with a simulator** — check the surface through a deuteranopia
   filter (or grayscale) before calling it done, the same way Miguel's rule is
   checked on computed layout, not just CSS source.

## Appears in

- [[ux-researcher]] — the accessibility persona this agent introduced (05/08);
  sibling to [[Miguel]] (touch) and Owner-Operator (readability)
- `usability-tester` agent — second default accessibility lens alongside Miguel
- Deviante monitoring — `MachineIllustration` visibility rebuild (luminance
  contrast, blue not green) and the green-reliance flag on monitoring identity
  / health and drift status coding (open owner decision)

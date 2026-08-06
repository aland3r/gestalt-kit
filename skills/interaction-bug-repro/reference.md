# Interaction bug repro

## Why this exists

21/07/2026: a UC row close-animation flicker was reported, code-reviewed,
"fixed", and verified via automated browser testing (Claude Browser MCP
driving a tab that wasn't the frontmost/visible one). The owner, testing
for real afterward, still saw a card blink after close — same session, a
*different* root cause (a read-view CSS rule sharing the fixed element's
name but never touched). The synthetic repro didn't catch it because the
polling showed a ~460ms CSS transition apparently completing in ~50-100ms
— a strong sign `requestAnimationFrame` wasn't firing at a normal cadence
in that tab, not evidence the interaction was smooth. The automated tool
had a real gap; the owner's eyes did not.

## The rule

**An owner-reported "I can see X" about motion/animation on a live surface
is ground truth — it overrides what an earlier automated/synthetic repro
showed.** Never answer a live bug report with "my test says it's fixed" as
if that settles it. When synthetic and real disagree, **trust the real
report** and keep digging — the tooling has a gap, not the owner.

## Process when the owner reports a live interaction bug

1. **Take the description literally.** What exactly did they see, and when
   — mid-animation, right after it finishes, on which action (open, close,
   cancel an edit, switch tabs)? Don't generalize to "probably the same
   flicker as last time" without checking the exact trigger they named.
2. **Reproduce with real interaction semantics, not just a scripted
   click+timer:**
   - Prefer a real click (accessibility-tree / coordinate click) over
     `element.click()` via `javascript_tool` — the latter can skip
     browser event-loop nuances a genuine pointer click goes through.
   - If screenshots are unavailable this session (a known recurring
     failure mode), fall back to sampled computed-style polling
     (`getBoundingClientRect`/`getComputedStyle` at several `setTimeout`
     delays after the trigger) — but treat suspiciously fast/clean numbers
     as a sign the tab isn't painting at a normal cadence, not proof the
     code is broken or fixed.
   - Use the currently **fronted, visible** tab. A backgrounded tab can
     throttle `requestAnimationFrame`, which breaks exactly the timing
     assumptions height/opacity transition code relies on.
3. **Distinguish the animation from what's mounted after it.** "Flickers
   after the animation" often isn't the open/close transition at all —
   check what swaps once the panel is already open and settled (editor ↔
   read view on cancel, for instance). Grep for a **second CSS rule with a
   near-identical name** targeting the sibling view (e.g. an edit-mode
   class fixed while its read-mode counterpart, same content, different
   selector, was never touched) — a background/style popping between two
   views of the same content reads exactly like a post-animation flicker.
4. **Fix, then ask the owner to re-check for real.** Don't mark a live
   interaction bug resolved from synthetic evidence alone. Say explicitly
   "I couldn't fully verify this live — please confirm" when auth/access
   blocks reproducing the exact state they saw (e.g. owner-only edit mode).

## Known false-negative traps (update this list when you find a new one)

- Backgrounded/non-focused automation tab → `requestAnimationFrame` doesn't
  fire at a normal cadence → CSS transitions appear to "complete" almost
  instantly in sampled polling, hiding jank a focused tab would show.
- `element.click()` bypasses some real-event nuances a genuine pointer
  click goes through — prefer a real accessibility-tree/coordinate click
  when the bug is specifically about *interaction feel*, not just a state
  check.
- Content-swap flicker (e.g. editor ↔ reader) can look identical to
  open/close flicker in a bug report — always verify *which* transition
  before touching animation code; the fix may be a plain CSS rule with no
  animation involved at all (as it was here — `.uc-editor__description`
  was de-gradiented, `.uc-folio__description` — same content, read-mode
  sibling — still had the full gradient/shine/glow, so the "flicker" was
  the background popping between the two on every edit-mode toggle).

## Related

- [interaction-designer.md](../../agents/interaction-designer.md) — owns
  this skill; its "Known hot spots" section is artifact-specific memory,
  this skill is the *process* memory for handling a report.
- [autolayout-ux.md](../../partials/autolayout-ux.md) — static spacing
  bugs (`ui-designer`'s domain) vs motion/feel bugs (this skill).

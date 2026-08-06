# Prefer existing files

**Every task:** before creating a new file, look for an existing home.

## Rule

1. Search for an existing markdown, skill, agent, schema, or module that
   already covers the topic.
2. **Update that file** (or a linked partial) instead of adding a sibling
   `*-v2.md`, `notes.md`, or parallel README.
3. Create a new file only when:
   - the owner explicitly asks for a new artifact, or
   - no reasonable existing file exists and the content would pollute an
     unrelated doc.
4. New shared rules go in `gestalt-kit/partials/` and are **linked**, not
   pasted into many skills.

## Why

Duplicates become drift. Truth Keeper and repo-consistency assume one SoT
per domain — extra files are replicas that rot.

## Related

- [partials/owner-preferences.md](../../partials/owner-preferences.md) — no clutter UI
- [repo-consistency.md](../repo-consistency/reference.md) — no junk names / scope
- [architecture.md](../../docs/architecture.md) — architecture SoT

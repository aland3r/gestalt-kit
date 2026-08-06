<!--
PARTIAL: tool-agnostic kit navigation + parsing contract.
Link from: truth-keeper, declutter, maestro, kit-entry skill, architecture.
Do not fork per IDE — adapters only mirror discovery paths into gestalt-kit/.
-->

# Kit navigation & agnostic parsing

**Contract:** every AI host (Cursor, Claude Code/Cowork, Antigravity, future)
loads **the same tree** from `gestalt-kit/`. Host-specific folders hold **thin
adapters** or **plugin manifests** — never a second copy of skill/agent bodies.

Situational quotas live in [ai-tooling.md](ai-tooling.md) (`maestro`). Structure
does not change when the host changes.

## Cold start (no chat history)

Read in order — stop when the task is clear:

| Step | Path | Why |
|------|------|-----|
| 1 | Repo root [`AGENTS.md`](../../AGENTS.md) | Hub entry for all hosts |
| 2 | [`gestalt-kit/README.md`](../README.md) | Mental model: skill / agent / partial / command |
| 3 | [`partials/kit-navigation.md`](kit-navigation.md) | This file — parsing + adapter rules |
| 4 | [`partials/sot-matrix.md`](sot-matrix.md) | One domain → one SoT |
| 5 | [`partials/active-scope.md`](active-scope.md) | IO + DV only; no Flashbrix |
| 6 | [`partials/system-requirements.md`](system-requirements.md) | Hard cross-cutting rules (language, stack boundaries) |
| 6b | [`partials/mcp-connectors.md`](mcp-connectors.md) | Which external connectors (Supabase, Notion, Figma, Drive) may be authorized this session, and their fallbacks |
| 6b | [`partials/portfolio-completion.md`](portfolio-completion.md) | Progress %, version label, portfolio-done = all UCs × four products |
| 7 | [`docs/architecture.md`](../docs/architecture.md) | Architecture + folder plant |
| 8 | Task skill `gestalt-kit/skills/<name>/` or agent `gestalt-kit/agents/<name>.md` | Domain work |

Optional command on session open: `/kit-entry` or load skill `kit-entry`.

**Do not** start from stub `doc/agents/`, chat memory, or IDE-global product
skills (e.g. Cursor `skills-cursor/*`) for Gestalt domain rules — those are
IDE features, not this repo's knowledge home.

## Host adapters (discovery only)

| Host | Adapter path | Rule |
|------|--------------|------|
| **Cursor** | `.cursor/skills/<name>/SKILL.md` | Thin pointer → `gestalt-kit/skills/<name>/`. Regenerate: `node scripts/sync-cursor-adapters.mjs` |
| **Claude Code** | `gestalt-kit/.claude-plugin/plugin.json` + `claude --plugin-dir gestalt-kit` | Optional local symlink `.claude/skills/gestalt-kit` → `gestalt-kit` (gitignored; see Activation in kit README) |
| **Cowork / other** | Same kit paths | No fork; copy plugin pattern or open repo at hub root |

**Editable SoT:** `gestalt-kit/` (+ live `portfolio.kit_docs` for site runtime).
**Never editable SoT:** `.cursor/skills/*` bodies beyond the thin template.

## Parse: skill folder

Path: `gestalt-kit/skills/<name>/`

```
1. Read SKILL.md YAML frontmatter → name, description, optional disable-model-invocation
2. If SKILL.md says "see reference.md" (or body is short) → read reference.md for full procedure
3. If command (disable-model-invocation: true) → run only when owner invokes /name
4. Follow links to partials/ — do not paste partial text into new files
5. Edit only under gestalt-kit/skills/<name>/ — then sync Cursor adapters if frontmatter changed
```

## Parse: agent file

Path: `gestalt-kit/agents/<name>.md`

```
1. Read YAML frontmatter → name, description, model, effort, skills: [ … ]
2. skills: list = kit skill **names** — load each from gestalt-kit/skills/<name>/
3. Body = persona mandate, boundaries, output format
4. Agents conduct work; they do not replace partials for shared one-line rules
5. Edit only gestalt-kit/agents/<name>.md
```

## Parse: partial

Path: `gestalt-kit/partials/<name>.md`

```
1. Optional HTML comment PARTIAL: … at top
2. Shared fragment — **link**, never paste wholesale into agents/skills/chat
3. Partials are the cheapest instrument (maestro): scope, SoT, tooling, vocabulary
4. Edit in place; do not create ai-tooling-v2.md parallel files
```

## Parse: vault & plans (replicas / temporary)

| Path | Role | SoT on conflict |
|------|------|-----------------|
| `gestalt-kit/vault/` | Obsidian UC prose, writing | Live `portfolio.use_cases` wins — [uc-esteira.md](uc-esteira.md) |
| `gestalt-kit/plans/` | Sprint day focus | Not quest SoT — live `portfolio.quests` |

Obsidian opens **`gestalt-kit/vault/`** only — not the whole kit.

## Declutter hunt (parallel copies)

| Location | Verdict |
|----------|---------|
| `.cursor/skills/*/SKILL.md` with full procedure body | **Restore thin adapter** (`sync-cursor-adapters.mjs`) |
| `.claude/worktrees/**/gestalt-kit` full duplicate | Flag / delete after owner yes |
| `.claude/skills/**` full mirrored skill trees | Delete; symlink or plugin-dir only |
| `doc/**` (qualquer coisa) | Delete — `doc/` foi removido em 22/07/2026, nem stub sobrou |
| `gestalt-vault/**` (qualquer coisa) | Delete — removido em 22/07/2026; vault = `gestalt-kit/vault/` |
| Obsidian notes pasting agent prompts | Move into kit; delete note copy |

## Truth-keeper checks

When asked “where is the skill?” or “which file is SoT?”:

1. Name domain → [sot-matrix.md](sot-matrix.md)
2. Authoring text → `gestalt-kit/`
3. Site runtime text → `portfolio.kit_docs` (MCP / `check-kit-drift.mjs`)
4. Adapter path → pointer only; if adapter body ≠ template → **drift**

## Maestro routing (instrument pick)

| Need | Create |
|------|--------|
| Shared rule / matrix / host table | **Partial** (`partials/`) |
| Procedure when doing X | **Skill** (`skills/<name>/`) |
| Owner-only side effect | **Command** (`disable-model-invocation: true`) |
| Persona + judgment + multi-step role | **Agent** (`agents/`) |
| Cold start without history | **Skill** `kit-entry` (this navigation partial is linked, not duplicated) |

Do **not** create `cursor-*` or `claude-*` skill names — extend the kit; adapters follow.

<!--
PARTIAL: current AI tooling context for maestro (and anyone routing work).
Owner updates when plans/limits/tools change — do not invent quotas.
-->

# AI tooling — current stack

**Principle:** Gestalt knowledge is **tool-agnostic**. Canonical home is
`gestalt-kit/` (skills, agents, partials). Cursor, Claude Code, Cowork,
Antigravity, or another host may load the same kit. Do not fork SoT per IDE.

**Bootstrap (no history):** `AGENTS.md` → `gestalt-kit/README.md` →
[kit-navigation.md](kit-navigation.md) → [sot-matrix.md](sot-matrix.md) — or
skill `kit-entry`.

**Practice:** decisions still need the **current** stack — which plans are
paid, which meters are exhausted, what the owner is testing — so maestro
can pick cheap routes and avoid dead ends.

## Owner snapshot (edit when it changes)

| Host / product | Plan / notes | Status (owner) | Updated |
|----------------|--------------|----------------|---------|
| Cursor | Paid | **usage limit reached** | 2026-07-20 |
| Claude (claude.ai / Code / Cowork — shared Pro meter) | Paid (2nd paid plan) | **usage limit reached** | 2026-07-20 |
| Antigravity | — | maybe testing later | — |
| Other | — | — | — |

When both Cursor and Claude are capped: prefer **local-only / low-effort**
work, deferred specialist sessions, or a third host the owner names — never
assume unlimited quota.

## What stays agnostic vs what is situational

| Agnostic (always true) | Situational (ask / read this partial) |
|------------------------|----------------------------------------|
| SoT map, architecture, ORCA, vault, Supabase | Which chat host has quota today |
| Agent roster & kit layout | Model/effort the host can still afford |
| Prefer existing files, no Flashbrix | Whether to wait vs switch tool |

## Sibling partial: MCP connectors

Which AI **host** has quota lives here. Which **MCP connectors** (Supabase,
Notion, Figma, Drive, ...) are authorized *this session* lives in
[mcp-connectors.md](mcp-connectors.md) — same situational spirit, separate
concern. Check that one before assuming a capability is missing.

## Maestro duty

1. Read this partial at session open.
2. If the snapshot looks stale or the owner mentions a new tool → **ask**
   and update this file (prefer-existing-files: edit here, don’t spawn
   `ai-tooling-v2.md`).
3. Factor limits into lineup (don’t schedule high-effort ORCA + schema in
   the same breath when meters are red).

# Gestalt Kit

**This folder is the single knowledge home** in the hub: agents, skills, commands,
partials, architecture, Obsidian vault, and sprint plans. Edit here.
Stubs at `doc/` and `gestalt-vault/` are redirects only.
`.cursor/skills/` holds thin adapters — regenerate via
`node scripts/sync-cursor-adapters.mjs`.

This repository is the deliverable source code of Gestalt v1.0, the first version of
the product hub designed and engineered solely by Alander Ávila with an agnostic agent team.


**Cold start:** [partials/kit-navigation.md](partials/kit-navigation.md) · skill `kit-entry`

**Obsidian:** open [`vault/`](vault/README.md) — not the whole kit.

Format: [Agent Skills](https://agentskills.io) for Claude Code / Cowork /
Cursor.

## Mental model

| Word | What it is | Where |
|---|---|---|
| **Skill** | `SKILL.md` + usually `reference.md` | `skills/<name>/` |
| **Agent** | Named persona / system prompt | `agents/<name>.md` |
| **Partial** | Shared fragment — link, don’t paste | `partials/` |
| **Architecture + folder plant** | System patterns & hub layout SoT | `docs/architecture.md` |
| **Vault** | UC markdown replicas, MOCs, writing (Obsidian) | `vault/` |
| **Plans** | Sprint + UC template | `plans/` |
| **Command** | Skill with `disable-model-invocation: true` | e.g. `skills/ship-quest/` |
| **Plugin** | This whole folder + `.claude-plugin/plugin.json` | `gestalt-kit/` |

## Layout

```
gestalt-kit/
├── .claude-plugin/plugin.json
├── docs/architecture.md     # architect owns (incl. folder plant)
├── partials/
├── skills/<name>/
│   ├── SKILL.md
│   └── reference.md
├── agents/
├── vault/                   # Obsidian root
├── plans/                   # sprint + UC-Template
└── README.md
```

See § Folder plant in [docs/architecture.md](docs/architecture.md).

## Team of agents

| Agent | Role |
|---|---|
| `maestro` | Conducts agents; asks strategy; audits skills/partials/commands structure |
| `product-manager` | Viability vs sprint plan, cost, tools, optional standards (MPS.BR…) |
| `truth-keeper` | SoT map + drift (incl. plant vs live dirs) |
| `architect` | Owns `docs/architecture.md` + folder plant |
| `ui-designer` | Style/contrast/spacing — raw ADS → product-styled UI |
| `interaction-designer` | Motion / open-close / flicker on live surfaces |
| `ooux` | ORCA facilitator |
| `content-strategist` | Publish/message strategy with OOUX + vault core content |
| `ux-writer` | Production copy from strategist brief (i18n, kit, UI, UC fields) |
| `persona-crafter` | Personas complementary to UCs |
| `researcher` | Persona-grounded gut check on a UI decision, on demand |
| `usability-tester` | Structured persona test against live layout (touch/spacing) |
| `polyrepo-shipper` | Commit/push across registered remotes |
| `declutter` | Delete replicas outside kit; keep one knowledge home |
| `data-guardian` | Verify live Supabase before deleting seeds; block secret leaks on deploy |
| `uc-scaffolder` | Draft ABP use cases |
| scaffolds | `database-integrations`, `deviante-backend`, `deviante-frontend` |

## Where to look (owner cheat-sheet)

| Question | Open |
|---|---|
| Edit an agent? | `gestalt-kit/agents/` |
| Edit a skill? | `gestalt-kit/skills/<name>/` |
| Product use cases? | Live DB `/cases` + `partials/uc-esteira.md` (vault = replica) |
| Sprint days? | `gestalt-kit/plans/sprint-plan-2026-07.md` |
| Folder layout SoT? | `gestalt-kit/docs/architecture.md` § Folder plant |
| Confused by stub `doc/` or old vault path? | Ask `declutter` · read `partials/kit-navigation.md` |
| No chat history / new host? | Skill `kit-entry` |
| Delete seeds / check deploy secrets? | Ask `data-guardian` |
| PIBITI report (later move)? | Still `deviante/docs/` — pointer only for now |

Active scope: **IO + DV only** — `partials/active-scope.md`. Never Flashbrix.

## Commands

`/ship-quest` — `skills/ship-quest/SKILL.md` (`disable-model-invocation: true`).  
`/uc-gate` — `skills/uc-gate/SKILL.md` — owner confirmation esteira before coding
(`partials/uc-esteira.md`).  
`/kit-depara` — `skills/kit-depara/SKILL.md` — kit↔Supabase diff (`partials/kit-depara.md`).  
`/audit-merges` — `skills/polyrepo-merge-audit/SKILL.md` — detect merge conflicts, orphaned commits, and mismerges across repos (`polyrepo-shipper` integration).  
`/gamifier-sync` — `skills/gamifier-sync/SKILL.md` — audit quest log against Supabase + truth-keeper + roadmap; detect stale/orphaned quests (`gamifier` integration).
`/debug-drift-detection` — `skills/debug-drift-detection/SKILL.md` — trace a Deviante IPDD/ADWIN run from graph scope through series construction, persistence, and replay; read-only unless invoked with `--fix`.

## Activation

**Claude Code**

```bash
claude --plugin-dir gestalt-kit
```

**Cursor** — open hub at `c:\gestalt`; skills load from `.cursor/skills/`
(thin adapters). After editing kit skills:

```bash
node scripts/sync-cursor-adapters.mjs
```

**Optional Claude symlink** (local, gitignored `.claude/`):

```bash
mkdir -p .claude/skills
ln -s ../../gestalt-kit .claude/skills/gestalt-kit
```

Or copy plugin folder once: `cp -r gestalt-kit .claude/skills/gestalt-kit`

## How to add a skill

1. `mkdir gestalt-kit/skills/{name}`
2. `SKILL.md` with `name` + `description` (trigger phrases)
3. Put the full body in `reference.md` in the **same** folder
4. Add a thin `.cursor/skills/{name}/SKILL.md` — run `node scripts/sync-cursor-adapters.mjs`
5. Do **not** recreate content under stub `doc/` or IDE-global skill folders

## How to add an agent

Copy `agents/uc-scaffolder.md` — set `description`, `skills`, boundaries.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

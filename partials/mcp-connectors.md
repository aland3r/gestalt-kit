<!--
PARTIAL: registry of session-dependent MCP connectors — link from maestro,
architect, gestalt-database, ooux, ui-designer, ux-engineer, uc-esteira.
Do not hardcode a tool's mcp__<uuid>__name from a past session — the UUID
is random per session; only the keyword below is stable.
-->

# MCP connectors — registry (situational, per-session)

Connectors are **authorized per session by the owner** — never assume one
is connected, never assume one is permanently absent either. This is the
same "situational vs agnostic" split as [ai-tooling.md](ai-tooling.md), for
external tool access instead of AI host/quota.

## Check fast, in this order (token economy)

1. **Scan the system-reminder deferred-tools list first** — a newly
   authorized connector's tool names appear there automatically, no call
   spent.
2. Not there? One `ToolSearch` call with the connector's keyword below —
   not a broad guess, not a retry loop.
3. Still nothing? The connector isn't authorized this session. Say so
   plainly and use the connector's **fallback** (below) or ask the owner —
   never fake the data it would have returned.

## Known connectors

| Connector | `ToolSearch` keyword | What it's for | Owning skill/agent |
|-----------|---------------------|----------------|---------------------|
| Supabase | `supabase execute_sql` (also `apply_migration`, `list_tables`) | Live DB reads/writes — UC esteira, schema corrections | `gestalt-database`, `uc-esteira` |
| Notion | `notion` | OOUX Masterclass DB, product ORCA Hubs | `ooux` |
| Figma | `figma` | Design context, screenshots, Figma Make, Code Connect | `ui-designer`, `ux-engineer`, `design-system` |
| Google Drive | `drive` | Search/read files in the owner's Drive | ad hoc — no owning agent yet |
| MCP registry (always available, not session-gated) | `select:mcp__mcp-registry__search_mcp_registry,mcp__mcp-registry__suggest_connectors` | Discover a connector not yet authorized, offer connecting it | `maestro` |

**When a brand-new connector shows up mid-session** (owner authorizes one
we've never seen): add a row here the same session, with its keyword and
which skill/agent should own it — don't let the next session rediscover it
from zero.

## Fallbacks when a connector is missing

- **Supabase down** → `gestalt-database` § "Ad-hoc read queries without
  MCP" (direct `pg` script, `deviante/api/.env` creds) — read-only; writes
  still need the owner to run them or the connector back.
- **Notion down** → ask the owner to paste the specific lesson/page
  content (masterclass `.vtt` → `data/masterclass-vtt/`, local-only,
  never commit) — never fabricate a masterclass quote.
- **Figma down** → ask the owner (see `sot-matrix.md` § "Figma is always
  the design reference") — never freehand a design from memory.

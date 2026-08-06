# Kit ↔ Supabase Sync Policy

**Owner:** `architect` agent + `truth-keeper` (SoT verification)

**Purpose:** Dados do gestalt-kit (agents, skills, partials, architecture, personas, plans) originários do repositório local devem ser sincronizados periodicamente para `portfolio.kit_docs` no Supabase, mantendo a verdade local como SoT.

## Sync triggers

### 1. Time-based (periodic)
- **Frequency:** Diariamente, entre 02:00 e 04:00 UTC (fora do horário de work normal)
- **Command:** `node scripts/sync-kit.mjs --schedule daily`
- **Owner approval:** Nenhuma (routine automática)

### 2. Change-based (commit threshold)
- **Trigger:** Após 10+ commits em `gestalt-kit/` em qualquer branch
- **Detection:** `git log --oneline origin/master.. gestalt-kit/ | wc -l`
- **Command:** `node scripts/sync-kit.mjs --force-update`
- **Owner approval:** `architect` agent revisa diffs, pede yes/no se conflito com DB

### 3. On-demand (owner request)
- **Command:** `/sync-kit [--force] [--dry-run]`
- **Approval:** Explícita do owner (yes/no no chat)

## What syncs

| Item | Source (local SoT) | Destination (Supabase) | Condition |
|------|---|---|---|
| **Agents** | `gestalt-kit/agents/*.md` | `portfolio.kit_docs` (type=agent) | If frontmatter changed or new agent |
| **Skills** | `gestalt-kit/skills/*/SKILL.md` + `reference.md` | `portfolio.kit_docs` (type=skill) | If `name`, `description`, or `reference.md` body changed |
| **Architecture** | `gestalt-kit/docs/architecture.md` | `portfolio.kit_docs` (type=doc, path='architecture') | If any section changed |
| **Partials** | `gestalt-kit/partials/*.md` | `portfolio.kit_docs` (type=partial) | If content changed |
| **Personas** | `gestalt-kit/vault/*/personas/*.md` | `portfolio.kit_docs` (type=persona) | If new or updated |
| **Plans** | `gestalt-kit/plans/sprint-plan-*.md` | `portfolio.kit_docs` (type=plan) | If plan content changed |
| **Vault MOCs** | `gestalt-kit/vault/**/README.md`, topic maps | `portfolio.kit_docs` (type=vault_moc) | If new MOC or structure changed |

## Conflict resolution

**Rule:** Local repo always wins (SoT).

If Supabase has conflicting content (someone edited via web UI instead of pushing to git):

1. **truth-keeper logs** the drift (file, hash mismatch)
2. **architect agent** reviews both versions
3. **Decision:**
   - If DB edit is recent (< 1 day) and not from git: ask owner to merge manually or discard DB version
   - If DB edit is stale: overwrite with local SoT
   - If both are substantive: pause sync, ask owner which wins

## Sync schema

Rows added/updated in `portfolio.kit_docs`:

```sql
INSERT INTO portfolio.kit_docs (
  path,           -- e.g. "agents/polyrepo-shipper" or "skills/uc-gate"
  type,           -- 'agent' | 'skill' | 'doc' | 'partial' | 'persona' | 'plan' | 'vault_moc'
  name,           -- from frontmatter.name
  description,    -- frontmatter.description (first line)
  content,        -- full markdown body
  git_hash,       -- sha of the file in git (for drift detection)
  synced_at,      -- now()
  synced_by,      -- 'kit-sync-policy' (automated) or 'owner-manual'
  metadata        -- JSON: {frontmatter, word_count, section_count, ...}
) VALUES (...)
ON CONFLICT (path, type) DO UPDATE SET
  content = EXCLUDED.content,
  git_hash = EXCLUDED.git_hash,
  synced_at = now(),
  metadata = EXCLUDED.metadata
WHERE portfolio.kit_docs.git_hash != EXCLUDED.git_hash;
-- Only update if git hash changed (avoid churn)
```

## Drift detection (truth-keeper)

Before each sync, verify:

```bash
# For each row in portfolio.kit_docs
for row in $(select path, type, git_hash from kit_docs); do
  local_hash=$(git hash-object gestalt-kit/$path)
  if [ "$local_hash" != "$row.git_hash" ]; then
    # Drift found: local changed but DB still has old version
    # Sync will fix it (local wins)
  fi
done
```

After sync, verify round-trip:
```bash
# Fetch the synced row back and compare
local_content=$(cat gestalt-kit/$path)
db_content=$(select content from kit_docs where path='...')
if [ "$local_content" != "$db_content" ]; then
  # Round-trip failed — log critical alert
fi
```

## Integration with other agents

### architect agent
- Owns the schema and naming of synced items (no item syncs without architect review first)
- Reviews diffs before sync on change-based trigger
- Updates `gestalt-kit/docs/architecture.md` as the authority on folder plant and patterns

### product-manager agent
- Flags if a sync would break a live feature (e.g., skill is used in production, cannot change signature)
- Approves breaking changes to skill/command signatures

### truth-keeper agent
- Detects and logs all drift events
- Raises alarm if round-trip fails
- Maintains audit log in `data/audit-logs/kit-sync-*.json`

### polyrepo-shipper agent
- Does NOT push to Supabase directly; only commits to git
- Triggers sync policy after successful git push to master (via hook)
- Reads sync status from Supabase to verify remote state matches local

## Commands

### Sync everything (dry-run)
```bash
/sync-kit --dry-run
```
→ Show what would be synced, no DB changes

### Force sync + approve
```bash
/sync-kit --force
```
→ Ask owner yes/no, then sync all changed items

### Sync one item
```bash
/sync-kit --item agents/polyrepo-shipper
```
→ Sync only that file

### View sync status
```bash
/sync-kit --status
```
→ Show last sync time, count of items, drift warnings

## Audit log

Each sync run logs to `data/audit-logs/kit-sync-YYYY-MM-DD-HH-mm-ss.json`:

```json
{
  "timestamp": "2026-07-22T03:45:12Z",
  "trigger": "time-based",
  "items_synced": 12,
  "items_skipped": 0,
  "drift_detected": 0,
  "conflicts": [],
  "git_hashes": {
    "agents/polyrepo-shipper": "abc123...",
    "skills/uc-gate": "def456..."
  },
  "synced_by": "kit-sync-policy",
  "status": "success"
}
```

## Checklist before deploying sync policy

- [ ] `portfolio.kit_docs` table exists and has `git_hash` column
- [ ] Architect reviews schema in `gestalt-kit/docs/architecture.md`
- [ ] truth-keeper verifies no existing drift in kit_docs (stale rows deleted if necessary)
- [ ] Dry-run on current master, confirm all items detected
- [ ] One manual sync approved by owner as smoke test
- [ ] Schedule set in CI/cron (daily 02:00 UTC)
- [ ] Slack alert webhook configured for drift/conflict events
- [ ] Audit log directory `data/audit-logs/` writable and backed up

## Related

- [SoT matrix](sot-matrix.md) — which data lives where
- [kit-navigation.md](kit-navigation.md) — git as SoT, Supabase as replica
- [architect.md](../agents/architect.md) — owner of kit structure
- [truth-keeper.md](../agents/truth-keeper.md) — drift detection

# Polyrepo Merge Audit

**Purpose:** Detect merge conflicts, orphaned commits, and mismerges when multiple Claude sessions edit the same repositories simultaneously. Runs before shipping or as a scheduled routine.

**Owner control:** `/audit-merges [--repos hub,deviante-api,deviante-web,portfolio] [--routine] [--fix]`

## Registered repos

| Path (under gestalt/) | Remote | Branch SoT | Session branches |
|---|---|---|---|
| `.` (hub) | `aland3r/gestalt-hub` | `master` | `claude/*` |
| `portfolio/` | `aland3r/portfolio` | `main` | `claude/*` |
| `deviante/api/` | `aland3r/deviante-api` | `main` | `claude/*` |
| `deviante/web/` | `aland3r/deviante-web` | `main` | `claude/*` |

## Detection heuristics

### 1. Unmerged branches (>0 commits, >1 day old)

**Command:** `git log {SoT-branch}..{branch} --oneline` for each local `claude/*` branch

**False-positive guard:**
- Only flag if branch has >0 commits ahead of SoT
- AND branch is >1 day old (skip fresh work-in-progress)
- AND last commit message does not start with `WIP` or `Draft`

**Action:** Merge to SoT or delete

---

### 2. Branch divergence (local vs remote)

**Command:**
```bash
git fetch origin
git rev-list --left-right --count {SoT-branch}...origin/{SoT-branch}
```

**False-positive guard:**
- Only flag if local is >0 commits ahead of remote
- AND no other session is currently active (heuristic: time since last remote commit <5 min = other session pushing)
- Consider unintentional force-push rebase — alert if divergence is >1 commit with same content hash

**Action:** 
- If local ahead: push
- If remote ahead: fetch + rebase (never force-push)
- If both diverged: flag as conflict, ask owner

---

### 3. Orphaned commits (not on origin/claude/*)

**Command:**
```bash
git log --all --graph --decorate --pretty=format:'%h %d %s'
# Compare each session branch against origin/claude/{same branch}
```

**False-positive guard:**
- Only flag if commit author email ≠ current session (different person or tool)
- Skip if commit is <1 hour old (still being worked on)

**Action:** Push to origin with `git push -u origin {branch}`

---

### 4. Conflicting file modifications (same file touched in 2+ active branches)

**Command:**
```bash
for branch in $(git branch -a --format='%(refname:short)' | grep claude/); do
  git diff {SoT-branch}..{branch} --name-only
done
# Compare outputs, flag files appearing 2+ times
```

**Overlap check:** For flagged files, run:
```bash
git diff {branch1}..{branch2} -- {file}
# Only flag if changes are in overlapping line ranges (not just same file)
```

**False-positive guard:**
- Ignore trivial conflicts: package.json, lock files, auto-formatted files
- Ignore if one branch is >2 days old (probably abandoned)

**Action:** Review both branches, pick one, cherry-pick non-conflicting chunks, or resolve manually

---

### 5. Stale untracked/deleted files (indicators of partial merges)

**Command:** `git status --porcelain`

**Age check:**
```bash
find . -type f -newermt '-3 days' ! -path '.git/*' ! -path 'node_modules/*'
# Cross-reference with untracked files from git status
```

**Ignore list:** `.log`, `.tmp`, `.err`, `node_modules/`, `dist/`, `build/`, `.next/`, etc. (see `repo-consistency.md`)

**False-positive guard:**
- Only flag files older than 3 days (fresh ignores don't matter)
- Only flag if they block a merge (e.g., in a conflicted path)

**Action:** Delete or add to `.gitignore`

---

### 6. Duplicate/reverted commits (same pattern appearing 2+ times)

**Command:**
```bash
git log --all --pretty=format:'%s' | sort | uniq -d
# For each duplicate, show which branches + timestamps
```

**False-positive guard:**
- Only flag if message is >10 chars (ignore `Fix typo`, `WIP`)
- Skip if both commits have >2 days between them (intentional re-do is valid)
- Skip if one is explicitly reverted (message contains `revert` or has `Revert` commit)

**Action:** Decide which one to keep, force one to be reverted, or keep if intentional

---

## Severity ranking

1. **Critical** — blocks all shipping:
   - Conflicting changes on same files in 2+ branches (overlap)
   - Remote ahead of local on main branch + local also has unpushed commits (divergence)
2. **High** — should merge before shipping:
   - Unmerged branches >3 days old with >5 commits
   - Orphaned commits on session branch not pushed to origin
3. **Medium** — review recommended:
   - Branch divergence (local ahead, safe to push)
   - Stale untracked files blocking a merge path
4. **Low** — FYI:
   - Unmerged branches <1 day old (WIP)
   - Duplicate commits with >2 days between them (intentional re-do)

---

## Output format

### Before acting

```
POLYREPO MERGE AUDIT
Scope: {hub, portfolio, deviante/api, deviante/web}

═══════════════════════════════════════════════════════════

CRITICAL ISSUES (blocks shipping):
  ✗ portfolio/main: remote 2 commits ahead, local 3 unpushed
    → git fetch + rebase + git push required before shipping

UNMERGED BRANCHES (>0 commits, >1 day old):
  ⊘ hub: claude/cranky-greider-a1ac5a (3 commits, 5 days old)
    Authors: self (Claude session)
    Last: "Add MCP connector registry partial" (21/07)
    → Suggest: git merge master (or delete if abandoned)

  ⊘ deviante/web: claude/uc-2-dev-validation-ityy56 (5 commits, 2 days old)
    Authors: self (Claude session)
    Last: "Complete UC2 spec validation" (19/07)
    → Suggest: git merge main

BRANCH DIVERGENCE (local ≠ remote):
  ⟷ deviante/api/main: local +3 commits | remote +0 commits
    → Suggest: git push origin main (safe)

CONFLICTING FILE EDITS (overlap in 2+ branches):
  ⚠ gestalt-kit/agents/architect.md:
    - Edited in: claude/cranky-greider-a1ac5a (lines 42–55)
    - Edited in: claude/funny-pare-3bbc22 (lines 40–50)
    → Overlap at line 42–50: MUST RESOLVE before merge
    → Suggest: Manual review, pick one, cherry-pick non-overlapping chunks

  ⚠ gestalt-kit/skills/deviante-domain/reference.md:
    - Edited in: claude/uc-15-alt-flows-ghjk23 (line 120)
    - Edited in: claude/uc-14-personas-qwer99 (lines 118–122)
    → Overlap: MUST RESOLVE before merge

ORPHANED COMMITS (not on origin/claude/*):
  ◆ hub: a7cfc6c "Route all interaction over MCP connectors"
    Local branch: claude/cranky-greider-a1ac5a
    Origin status: NOT FOUND
    → Suggest: git push --set-upstream origin claude/cranky-greider-a1ac5a

STALE UNTRACKED FILES (may block merge):
  ? hub/_dev-server.err.log (3 days old)
    → Suggest: delete or add to .gitignore

DUPLICATED COMMITS (same message on 2+ branches):
  ≈ "Update Deviante schema with drift detection":
    - Commit abc1234 on claude/drift-schema-old (5 days ago)
    - Commit def5678 on claude/drift-schema-new (1 day ago)
    → Suggest: delete old branch or revert the earlier commit

═══════════════════════════════════════════════════════════

SUMMARY:
  Critical: 1 (divergence on portfolio/main)
  High: 3 (unmerged branches + orphaned)
  Medium: 2 (conflicting files)
  Low: 2 (stale files + duplicates)

SHIPPING BLOCKERS:
  ✗ Resolve portfolio/main divergence (git fetch + rebase)
  ✗ Resolve conflict in gestalt-kit/agents/architect.md
  ✗ Merge or delete unmerged branches (3 recommendations)

RECOMMENDATION:
  1. Run: git fetch (all repos)
  2. Run: git rebase origin/main (on portfolio/main if local ahead)
  3. Review: gestalt-kit/agents/architect.md (fix overlap)
  4. Run: git merge master (on hub/claude/cranky-greider-a1ac5a)
  5. Run: git push -u origin {all session branches}
  6. Run: /audit-merges again to verify

After cleanup → polyrepo-shipper can safely proceed.
```

### After acting (--fix)

```
AUDIT COMPLETE — CLEANUP APPLIED

✓ portfolio/main: rebased, 3 commits pushed
✓ hub: merged claude/cranky-greider-a1ac5a → master, pushed
✓ gestalt-kit/agents/architect.md: conflict resolved (kept cranky-greider version)
✓ Deleted stale file: _dev-server.err.log
✓ Pushed orphaned commits: a7cfc6c on claude/cranky-greider-a1ac5a

REMAINING:
  ⊘ deviante/web: claude/uc-2-dev-validation-ityy56 still unmerged (2 commits)
    → Needs manual review before merge

Next: /audit-merges --routine to run background check.
```

### Routine mode (--routine)

**If no issues:**
```
✓ All repos clean. No shipping blockers.
```

**If issues:**
```
⚠ 2 issues found (medium severity). Run `/audit-merges` for details.
  - 1 unmerged branch >2 days old
  - 1 stale untracked file
```

---

## Integration with polyrepo-shipper

The `polyrepo-shipper` agent calls this audit automatically before pushing if any of these conditions are met:

1. Owner runs `/ship-quest` or commit/push workflow
2. >1 repo is dirty at the same time (multi-repo ship)
3. Session branch has history vs origin (orphaned commits possible)

**polyrepo-shipper protocol:**
```
1. Run: /audit-merges (auto-detect scope)
2. If critical issues: STOP, report to owner, ask for fix
3. If high issues: REPORT, wait for owner approval
4. If medium/low: REPORT, ship if owner says yes
5. Execute shipping plan after audit passes
```

---

## Routine operation (scheduled)

When `/audit-merges --routine` is set to run hourly (during active sessions):

- **Frequency:** Every 1 hour (or configurable interval)
- **Silence after clean:** If 3 consecutive runs show no issues, suppress output until next issue found
- **Escalation:** If critical issues detected, notify owner immediately (Slack / console alert)
- **Log:** Store findings in `data/audit-logs/` (JSON per run, indexed by timestamp)

Example cron / schedule:
```
0 * * * * /audit-merges --routine
```

---

## Command reference

### Manual audit (verbose)
```
/audit-merges
```
→ Scans all repos, reports all issues, no action

### Audit specific repos
```
/audit-merges --repos hub,portfolio
```
→ Only hub and portfolio

### Auto-fix with approval
```
/audit-merges --fix
```
→ Proposes fixes, waits for yes, applies (safe merges, rebases, pushes)

### Routine background check
```
/audit-merges --routine
```
→ Run hourly; silent if clean; alert if issues found

### Combine flags
```
/audit-merges --repos deviante-api,deviante-web --routine
```
→ Routine check on Deviante repos only

---

## False positives — known safe ignores

- `package-lock.json` / `pnpm-lock.yaml` / `yarn.lock` (auto-formatted)
- `.next/`, `dist/`, `build/`, `node_modules/` (generated)
- `portfolio.db`, `*.log`, `.DS_Store`, `*.swp` (temp)
- Commits in `.cursor/skills/` synced from kit (auto-generated, divergence OK)
- `CHANGELOG.md` (append-only, safe to merge)

---

## Limitations & future work

- Does not detect silent data corruption (conflicting DB migrations)
- Does not verify commit signatures
- Heuristic on "other session is pushing" is time-based (unreliable across timezones)
- Could integrate with GitHub API to check PR/CI status (not yet)
- Could store audit history (trend detection: is divergence getting worse?)

---

## See also

- [polyrepo-shipper.md](../agents/polyrepo-shipper.md) — shipping protocol
- [repo-consistency.md](../repo-consistency/reference.md) — which files/names are allowed
- [gestalt-context/reference.md](../gestalt-context/reference.md) — registered repo list (SoT)

---
name: polyrepo-merge-audit
description: >-
  Audit polyrepo (hub, deviante/api, deviante/web, portfolio) for merge conflicts, orphaned commits, and mismerges from concurrent sessions. Use before shipping or as a scheduled routine.
disable-model-invocation: true
argument-hint: "[--repos hub,deviante-api,deviante-web,portfolio] [--routine] [--fix]"
---

<!--
Command: only the owner invokes (/audit-merges). Detects merge issues in multi-session environment.
Full contract: gestalt-kit/skills/polyrepo-merge-audit/reference.md
-->

Run the **polyrepo merge audit** to detect unmerged branches, orphaned commits, divergence, and conflicting changes across all registered repos.

1. Read [gestalt-kit/skills/polyrepo-merge-audit/reference.md](reference.md) for full detection logic.
2. **Inventory phase** — for each repo (or `--repos` subset), run git status/log/diff to classify dirty state.
3. **Report findings** — unmerged branches, divergence, orphaned commits, conflicting file edits, stale untracked files. Rank by severity.
4. **Recommend actions** — merge, push, rebase, delete, or clean.
5. If `--fix` is passed, ask for approval before applying suggestions; if `--routine` is passed, suppress output unless issues found.
6. **Output** — see reference.md § Output Format.

**Note:** This command integrates with `polyrepo-shipper` — it runs automatically before shipping if conflicts are detected, but owner can also invoke manually with `/audit-merges [--repos ...] [--routine] [--fix]`.

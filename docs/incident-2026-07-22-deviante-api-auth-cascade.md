# Incident Report: Deviante API Auth 401 Cascade (2026-07-22)

**Status:** Resolved  
**Duration:** ~2 hours of session debugging  
**Root Cause:** 4 distinct bugs, each masking the next  
**Severity:** Blocking (dashboard inaccessible, 0 processes shown, all /api calls returned 401)  

## Summary

Dashboard login succeeded, but every API call (manager profile, list processes) returned `"Sessão inválida ou expirada."` 401. Four separate bugs layered on top of each other, each one invisible when the ones below it were failing first.

## The 4-Bug Stack

| # | Bug | Symptom | Root Cause | Fix |
|---|-----|---------|-----------|-----|
| **1** | `fly.toml` override | `gradlew: No such file or directory` on boot | `[build] image = "eclipse-temurin:21-jdk"` told Fly to ignore the Dockerfile and pull a bare JDK image with no source code | Removed `[build]` section from fly.toml; changed `[processes] api` to run the pre-built jar |
| **2** | OOM crash loop | Machine restarted 10 times, health checks intermittently failed | JVM in 256 MB container + HikariCP + Ktor + HTTP client for Supabase verification = OOM killer | Scaled VM to 512 MB |
| **3** | Malformed database secret | `Driver org.postgresql.Driver claims to not accept jdbcUrl` on boot | User pasted `DATABASE_JDBC_URL=jdbc:postgresql://...` (with the key name) instead of just the value into Fly dashboard | Re-set via `flyctl secrets set` with the correct bare value |
| **4** | Silent JSON parsing failure (THE REAL BUG) | `"Sessão inválida ou expirada."` on every valid token | Kotlin's `SupabaseAuthClient` used `json()` with default `ignoreUnknownKeys = false`. Supabase's `/auth/v1/user` response carries fields we don't model (aud, role, app_metadata, identities, etc.). SerializationException was swallowed by bare `catch (_: Exception) { null }`, making every valid token look invalid | Enabled `ignoreUnknownKeys = true` and added real logging (error when config missing, warn with status+body when Supabase rejects, error on exception caught) |

## Why Each Bug Was Invisible

**Bug #1 (fly.toml)**: The machine entered a crash-restart loop before ever getting to Gradle. The errors were buried in flame output. `flyctl logs --no-tail` showed `/gradlew: No such file or directory` but it looked like a permissions issue, not a deployment config problem.

**Bug #2 (256 MB)**: Once we rebuilt with the correct Dockerfile, the machine started but the JVM was OOM-killed after a few requests. Logs showed `Out of memory: Killed process (java)`, easy to spot once you looked. But the repeated restarts made the box flaky enough that every auth request was a race condition.

**Bug #3 (DATABASE_JDBC_URL)**: The secret had the **key name baked into the value**, so the driver saw `DATABASE_JDBC_URL=jdbc:postgresql://...` as a malformed URL. Clear error in logs, but it only appeared after #2 was fixed (512 MB = machine stable enough to consistently fail on the JDBC URL instead of OOM).

**Bug #4 (JSON parsing, the real one)**: The token verification was failing silently inside a try-catch that logged nothing. The backend sent back `401 "Sessão inválida ou expirada."` for every login, which looked like an auth config problem. But the config was fine — Supabase was responding, we just weren't parsing the response. Only visible after killing bugs 1–3 and adding real logging.

## Detection Failures

### What Should Have Been Obvious Earlier

1. **fly.toml override logic**: The `[build]` section in Fly's config takes priority over the Dockerfile. This is a known Fly.io gotcha, not an edge case. Should have been the first thing to check.
2. **Memory headroom for JVM**: A JVM with HikariCP + Ktor + HTTP client on 256 MB is guaranteed to fail under load. Rule of thumb: minimum 512 MB for production Ktor apps. We documented this in README but didn't validate on deploy.
3. **Secret format in UI**: Pasting `KEY=value` into Fly's "value" field (instead of just `value`) is a common user error on secret dashboards everywhere. The API should reject this format, but at minimum the CLI tool (`flyctl secrets set`) should warn when a value contains `=` near the start.
4. **Silent catch blocks**: The bare `catch (_: Exception) { null }` pattern in auth code is an anti-pattern. Every authentication path should log its failures.

### Scanning Techniques That Failed

- **"Check the error message"**: The error message was intentionally generic. It didn't say "failed to parse JSON from Supabase" — it said "session expired", which is correct from the user's perspective but hides the real failure.
- **"Test with fake token"**: We did this, got 401 as expected. But we couldn't tell if it was a config problem or a parsing problem.
- **Assuming each layer's correctness**: We trusted that if the frontend config was right and the backend was "running", the integration was good. Didn't interrogate the middle path (token verification) until it was the last thing left.

## Lessons for Gestalt v1.0

### For `debugger` Agent

- **Logging as debuggability**: Add instrumentation BEFORE shipping. Auth code especially needs real error logging, not just exceptions swallowed.
- **Layered failures**: When a single observable symptom (401) could come from 4 different layers, test each layer in isolation first, don't assume lower layers.
- **Configuration shadowing**: Audit all places where config can be overridden (Dockerfile vs. fly.toml, environment secrets vs. defaults, etc.) and ensure precedence is clear and documented.

### For `developers` Agent

- **Kotlin + JSON serialization**: Always use `ignoreUnknownKeys = true` for external APIs that may add fields later.
- **JVM sizing**: Never deploy a production JVM app with less than 512 MB. Measure actual heap use locally first.
- **Secret management on Fly**: Use `flyctl secrets set KEY=value` (with = in the command, but value is still just the value part), not the dashboard. The dashboard is error-prone for multi-line or special-char values.

### For `architect` Agent

- **Error handling in auth paths**: Replace bare `catch` blocks with structured logging. Consider a custom exception type for "auth verification failed" that distinguishes:
  - Missing/unconfigured secrets (config error)
  - Backend rejected the token (bad token)
  - Backend responded but parsing failed (serialization error)
- **Deployment config as code**: fly.toml shouldn't have a `[build]` section if a Dockerfile exists. The presence of BOTH is confusing. Prefer Dockerfile-only.
- **Configuration validation on startup**: Log all secrets' presence/absence (not their values) at app startup, so misconfig is obvious in the first few log lines.

### For `maestro` Agent

Create a new **skill `diagnose-auth-failures`** that:
1. Tests token verification in isolation (curl to `/auth/v1/user` with a known-good token)
2. Verifies JSON parsing by logging the raw response body
3. Checks all auth-related environment variables are set (even if we can't check their correctness)
4. Suggests "add logging here" at any `catch` block in auth code

Create a new **command `/audit-config-precedence`** that:
1. Scans for files that might override each other (Dockerfile + fly.toml, .env + application.yaml, etc.)
2. Flags hidden precedence rules (like fly.toml `[build]` overriding Dockerfile)
3. Suggests a single source of truth per configuration domain

### For `declutter` Agent

- **Gestalt v1.0 invariant**: All agents, skills, and commands MUST live in `gestalt-kit/` (agents/, skills/, commands/). Anything in `doc/` or `gestalt-vault/` is a redirect or historical artifact, not live.
- **Audit**: Scan for any `.md` files outside `gestalt-kit/` that describe workflows, agents, or skills. If found, they're either stale or duplicate — consolidate into `gestalt-kit/` and update links in AGENTS.md.
- **Single source of truth**: `gestalt-kit/README.md` + `gestalt-kit/AGENTS.md` list every agent/skill/command. Anything not listed there doesn't exist from a workflow perspective, even if the file exists in the repo.

## Timeline

| Time | Event |
|------|-------|
| T+0 | Dashboard shows 401 "Sessão inválida ou expirada." and empty process list |
| T+15m | Diagnosed fly.toml `[build]` override, fixed, redeployed |
| T+30m | Machine still crashing on restarts; scaled to 512 MB |
| T+45m | DATABASE_JDBC_URL had `KEY=` baked in; corrected via flyctl |
| T+90m | Added `ignoreUnknownKeys = true` to JSON parsing, deployed |
| T+105m | Dashboard now shows 4 demo processes, `/api/processes` returns 200 with data |

## Verification

- ✅ `deviante-api.fly.dev/ping` → 200 pong
- ✅ `deviante.alander.io/dashboard` → 4 demo processes visible (after login)
- ✅ `/api/manager/me` → 200 with manager profile
- ✅ `/api/processes` → 200 with process list
- ✅ Logs show successful Supabase token verification (no more generic "session expired" errors)

## Related Files

- [Routing.kt](../../../deviante/api/src/main/kotlin/Routing.kt) — L33 error response
- [SupabaseAuth.kt](../../../deviante/api/src/main/kotlin/SupabaseAuth.kt) — Fixed L30–60
- [fly.toml](../../../deviante/api/fly.toml) — Fixed: removed `[build]` section
- [README.md](../../../deviante/api/README.md) — Documents correct DATABASE_JDBC_URL format
- [vercel.json](../../../deviante/web/vercel.json) — Proxy rule `/api/*` → fly.io backend

---

**For Future Sessions:** This incident is now part of Gestalt's institutional knowledge. If you encounter another "401 on every auth request", check this doc first — the diagnostic path (layer isolation, logging addition) is the template.

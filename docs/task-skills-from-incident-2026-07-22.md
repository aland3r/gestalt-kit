# New Skills Needed: Auth Diagnostics & Config Audit (from 2026-07-22 Incident)

**Context:** See [incident-2026-07-22-deviante-api-auth-cascade.md](./incident-2026-07-22-deviante-api-auth-cascade.md) and [kit-governance-audit-2026-07-22.md](./kit-governance-audit-2026-07-22.md).

**Outcome:** Three new skills + one command to prevent this class of issue recurring.

---

## Skill 1: `diagnose-auth-failures`

**Location:** `gestalt-kit/skills/diagnose-auth-failures/`  
**Purpose:** Quick isolation of token verification failures (vs. config errors vs. parsing errors)  
**When to invoke:** Any 401 on an authenticated route  

### SKILL.md Outline

```
# Diagnose Auth Failures

Isolate whether a 401 "session expired" is caused by:
1. Missing/misconfigured secrets (SUPABASE_URL, SUPABASE_ANON_KEY, DATABASE_JDBC_URL, etc.)
2. Backend rejecting the token (bad JWT, expired, wrong scope)
3. Backend receiving and parsing the token incorrectly (serialization, type mismatch)

## Steps

1. **Check secrets presence** (not values)
   - Log all auth-related env vars to stdout (grep logs for "SUPABASE", "DATABASE")
   - Flag if any are blank or malformed (e.g., contain `KEY=value` instead of just `value`)

2. **Test Supabase connectivity**
   - `curl $SUPABASE_URL/auth/v1/settings` with `-H "apikey: $SUPABASE_ANON_KEY"`
   - If 200: Supabase is reachable and the key is accepted
   - If 403: The key is invalid or revoked
   - If timeout: Network issue, not auth

3. **Test token verification with a known-good token**
   - If you have a valid session locally, extract the access_token
   - `curl $SUPABASE_URL/auth/v1/user -H "Bearer $TOKEN" -H "apikey: $SUPABASE_ANON_KEY"`
   - Expected: 200 with JSON user object containing `id`, `email`, `user_metadata`, plus many other fields
   - If 401/403: Token is bad (expired, wrong format, etc.)

4. **Check JSON parsing strictness**
   - Look for `ignoreUnknownKeys` in the Kotlin/JS code doing the parse
   - If missing or false: Supabase responses will fail to deserialize (expected 8+ fields that aren't modeled)
   - Solution: Enable `ignoreUnknownKeys = true` or model all expected fields

5. **Add real logging**
   - Replace bare `catch (_: Exception)` with `logger.error("cause", exception)`
   - Include response status + body in logs: `logger.warn("Supabase rejected token: {} {}", status, body)`
   - This makes the next failure visible instead of silent

## Outcome

Report:
- ✅ or ❌ Secrets configured
- ✅ or ❌ Supabase reachable
- ✅ or ❌ Token valid
- ✅ or ❌ Token parsing configured correctly
- Suggested fix based on which step failed
```

### reference.md Outline

```
# Diagnose Auth Failures — Reference

Applies to: Kotlin Ktor + Supabase, React/TypeScript + Supabase Auth

## Common patterns that look the same but have different causes

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| 401 on every request, even with valid login | Parsing strictness (ignoreUnknownKeys false) | Set ignoreUnknownKeys = true |
| 401 on every request, with timeout in logs | Supabase network unreachable | Check SUPABASE_URL and firewall |
| 401 on first login, then 200 | Token verification works after first attempt (caching race) | Unlikely; check logs |
| 401 on profile endpoint only | Route doesn't attach token to request | Check auth header forwarding |
| 401 with cryptic "session expired" + no logs | Silent exception in catch block | Add logging to catch block |

## Checklist before shipping auth code

- [ ] Token verification logs both success and failure paths
- [ ] JSON parsing uses `ignoreUnknownKeys = true` for external APIs
- [ ] Secrets are logged at startup (names only, not values)
- [ ] Exception in auth code is logged with full detail, not swallowed
- [ ] Token test endpoint exists (`GET /api/manager/me` or equivalent) for debugging
```

---

## Skill 2: `audit-config-precedence`

**Location:** `gestalt-kit/skills/audit-config-precedence/`  
**Purpose:** Detect configuration shadowing and override conflicts  
**When to invoke:** Before any deployment, or when adding a new config source (Dockerfile, fly.toml, .env, application.yaml, etc.)  

### SKILL.md Outline

```
# Audit Config Precedence

Find hidden configuration overrides that may silence or contradict Dockerfile, environment, or code defaults.

## Sources that can shadow each other

- `Dockerfile` (build instructions)
- `fly.toml` (Fly.io deployment config, including [build] section)
- `.env` / `.env.production` (environment variables)
- `application.yaml` / `application.properties` (Ktor/Spring config)
- Environment secrets on platform (Fly, Vercel, GitHub Actions)
- Hardcoded defaults in code

## Common precedence bugs

| File | Precedence Issue | Example |
|------|------------------|---------|
| fly.toml `[build]` section | **Overrides Dockerfile entirely** if present | Setting `image = "ruby:2.7"` causes builder to ignore your Dockerfile |
| Env secrets | Override `[build]` in fly.toml | `FLY_BUILDPACKS` env var overrides buildpacks in fly.toml |
| `.env` local | Overrides `.env.production` if it exists alongside | Dev accidentally commits `.env` with test values |
| `application.yaml` env substitution | `${VAR:default}` means "use VAR or fall back to default" | If VAR is unset, default is used silently — hard to debug |

## Audit steps

1. **List all config files** in the project (find . -name "{Dockerfile,fly.toml,.env*,application.yaml,package.json}")
2. **Check each file for override patterns:**
   - fly.toml: look for `[build]` section (overrides Dockerfile)
   - application.yaml: look for `${...}` substitutions (hidden precedence)
   - Dockerfile: look for `ENV` directives (can be shadowed by platform secrets)
   - package.json: look for `predev`/`prebuild` hooks (run before actual build)
3. **Flag conflicts:**
   - "Dockerfile exists but fly.toml [build] also set — Dockerfile will be ignored"
   - "ENV DATABASE_URL in Dockerfile but also set as platform secret — platform secret wins"
   - ".env and .env.production both exist — unclear which is used locally"
4. **Suggest consolidation:**
   - "Move all Fly config to Dockerfile; remove [build] from fly.toml"
   - "Use platform secrets, not .env files, for sensitive values"
   - "Document precedence in README: Dockerfile < fly.toml < platform secrets < code defaults"
```

### reference.md Outline

```
# Audit Config Precedence — Reference

## When to run

- Before any deploy to Fly, Vercel, GitHub, or other platform
- When adding a new config file or platform
- Before shipping a project template

## Tools

- `flyctl config show` — shows fly.toml parsed with secrets applied
- `grep -r "^\[build\]" .` — finds all [build] sections that might override
- `grep -r "\${" application.yaml` — finds all env substitutions

## Example: fixing a precedence bug

**Problem:** fly.toml has `[build] image = "node:20"` and a Dockerfile exists. Fly ignores the Dockerfile.

**Fix:** Remove the `[build]` section from fly.toml entirely, let Fly auto-detect and use Dockerfile.

**Verify:** `flyctl config show | grep -A5 "^\[build\]"` should return empty.
```

---

## Skill 3: `enforce-logging-standards`

**Location:** `gestalt-kit/skills/enforce-logging-standards/`  
**Purpose:** Ensure all auth/error paths have real logging, not silent catches  
**When to invoke:** During code review of any auth, deployment, or error-handling code  

### SKILL.md Outline

```
# Enforce Logging Standards

Replace silent exception handling with structured, actionable logs.

## Rule 1: No bare catch blocks in auth code

❌ BAD:
\`\`\`kotlin
try {
  val response = client.get(url)
  parseResponse(response)
} catch (_: Exception) {
  null
}
\`\`\`

✅ GOOD:
\`\`\`kotlin
try {
  val response = client.get(url)
  parseResponse(response)
} catch (e: Exception) {
  logger.error("Failed to fetch from $url", e)
  null
}
\`\`\`

## Rule 2: Log both success and failure

Success logs help confirm "everything is working as intended", not just "no error occurred".

❌ BAD:
\`\`\`kotlin
if (token == null) {
  respond(401, "Session invalid")
}
\`\`\`

✅ GOOD:
\`\`\`kotlin
val user = verify(token)
if (user == null) {
  logger.warn("Token verification failed: token=$token")
  respond(401, "Session invalid")
} else {
  logger.debug("Token verified for user ${user.email}")
}
\`\`\`

## Rule 3: Log the full context, not just the error message

❌ BAD: `logger.error("Failed")`  
✅ GOOD: `logger.error("Failed to verify Supabase token: {} {}", response.status, response.bodyAsText())`

## Rule 4: Use appropriate log levels

| Level | Use Case |
|-------|----------|
| ERROR | Auth failed, secrets missing, network unreachable — user should know |
| WARN | Token rejected by Supabase, but this is expected for bad tokens — log it so we can debug if repeated |
| INFO | App starting, config loaded, schema migrated — operational milestones |
| DEBUG | Token verified successfully, manager profile fetched — not usually needed in prod |

## Audit checklist

- [ ] No `catch (_: Exception) { return null }` in code
- [ ] Every auth path (success, bad token, network error, parsing error) has a log statement
- [ ] Logs include full context (status, body, exception) not just "failed"
- [ ] Log levels match severity (ERROR > WARN > INFO > DEBUG)
```

---

## Command: `/audit-config-precedence`

**Location:** `gestalt-kit/commands/audit-config-precedence.md` (or in a commands registry)  
**Purpose:** Run the skill interactively and report findings  
**Invocation:** `/audit-config-precedence [path]` (defaults to current dir)  

### Definition Outline

```
# Command: /audit-config-precedence

Scan project for config shadowing and precedence bugs.

## Usage

\`\`\`
/audit-config-precedence                    # audit current dir
/audit-config-precedence /path/to/project   # audit specific dir
/audit-config-precedence --fix              # fix common issues (prompt for each)
\`\`\`

## Output

```
✅ Dockerfile exists (no [build] override in fly.toml)
✅ No .env and .env.production conflict
⚠️  fly.toml: [build] image = "..." found (will override Dockerfile)
⚠️  application.yaml: ${DATABASE_URL} substitution found (unclear which env var is used)
❌ Missing: secrets documented (add README section on which env vars are required)

Recommended fixes:
1. Remove [build] from fly.toml — use Dockerfile instead
2. Document DATABASE_URL precedence in README
\`\`\`

## Wires into

- `maestro` agent for orchestration
- `architect` agent for deployment review
- CI/CD pipelines (can run as pre-deploy gate)
```

---

## Implementation Checklist

**For maestro agent:**

- [ ] Create `gestalt-kit/skills/diagnose-auth-failures/` with SKILL.md + reference.md
- [ ] Create `gestalt-kit/skills/audit-config-precedence/` with SKILL.md + reference.md
- [ ] Create `gestalt-kit/skills/enforce-logging-standards/` with SKILL.md + reference.md
- [ ] Create `gestalt-kit/commands/audit-config-precedence.md` or registry entry
- [ ] Update `gestalt-kit/AGENTS.md` to list the 3 new skills
- [ ] Run `node scripts/sync-cursor-adapters.mjs` to generate `.cursor/skills/` adapters
- [ ] Test each skill by simulating an auth failure (e.g., breaking SUPABASE_URL)
- [ ] Add to `gestalt-kit/partials/active-scope.md` under "Debugging & Audit" section

**For declutter agent:**

- [ ] Add audit entry: "Verify new skills are listed in AGENTS.md" (3 new ones as of 22/07)
- [ ] Add task: "Weekly check that `.cursor/skills/` adapters are in sync with `gestalt-kit/skills/`"

---

## Related

- [incident-2026-07-22-deviante-api-auth-cascade.md](./incident-2026-07-22-deviante-api-auth-cascade.md) — The incident that revealed the need for these skills
- [kit-governance-audit-2026-07-22.md](./kit-governance-audit-2026-07-22.md) — Audit confirming kit structure is correct

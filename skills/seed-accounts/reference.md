# Seed accounts (Gestalt + Supabase)

**Auth:** Supabase Auth (hosted). **Data:** Supabase PostgreSQL, schema per product (`deviante`, `milebrick`, `harpia`, `portfolio`).

## Rule

**Users are created in Supabase Auth; the owner invites or creates testers in the dashboard.** App profile rows (e.g. `deviante.managers`) use the **same UUID** as `auth.users.id`. No self-service register UI in v1 dev.

## What to ship

| Layer | Do | Don't (v1 dev) |
|-------|-----|----------------|
| **Supabase Auth** | E-mail/password + Google OAuth via `@supabase/supabase-js` | Custom bcrypt login in Ktor |
| **DB profiles** | SQL seed / migrations for `managers`, product tables — `user_id` = auth UUID | `password_hash` in app tables |
| **API** | JDBC → Supabase Postgres; business data only | Store passwords in `deviante.users` |
| **Web** | Login via Supabase; `.env` with URL + anon key | Register page (redirect to login) |
| **Ops** | Create users in Supabase dashboard; share credentials | Open public sign-up |

## Gestalt access flow (Portfolio hub)

1. **Portfolio** (`alander.io`) — login Google → owner auto-bootstrap for `design@alander.io` and `alanderavila@gmail.com` (also overridable via `NEXT_PUBLIC_GESTALT_OWNER_EMAILS`).
2. **`/product`** — lista produtos permitidos; link abre landing em cada subdomínio.
3. **Sessão compartilhada** — cookie `.alander.io` via `@gestalt/auth` (SSO entre portfolio e apps).
4. **Sem permissão** — `/request-access` no portfolio; owner aprova em `/admin`.
5. **App landing** — botão **Continuar**: autenticado + `portfolio.product_access` → dashboard; senão login ou `/no-access`.

DDL: `data/schema/portfolio/` + `grants.sql` (RLS). Apply in Supabase SQL editor.

## Adding a tester (via admin)

1. Tester faz login Google no portfolio → **Solicitar acesso** (`/request-access`).
2. Owner abre **`/admin`** → aprova solicitação (marca produtos).
3. App provisiona `{schema}.users` + perfil mínimo automaticamente.
4. Tester abre produto em `/product` → **Continuar** → dashboard.

Manual owner/profile rows: insert in **Supabase** (Auth UUID = `portfolio.users.id`
/ `deviante.managers.user_id`). There is no `data/seed/` dump — live DB is SoT.
Vault → runtime UCs: `node scripts/sync-vault.mjs` with `DATABASE_URL`.

To wipe testers: delete in Supabase Auth dashboard + matching profile rows.

## Secrets

| App | File | Keys |
|-----|------|------|
| Portfolio | `portfolio/.env.local` | `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `NEXT_PUBLIC_GESTALT_OWNER_EMAIL` |
| Deviante web | `deviante/web/.env` | `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` |
| Milebrick web | `milebrick/web/.env` | same Supabase project |
| API | env / IDE run config | `DATABASE_JDBC_URL`, `DATABASE_PASSWORD` |

Never commit `.env`. Use `.env.example` as template.

## Supabase Auth — one project, all apps

**Authentication → URL Configuration**

| Field | Value |
|-------|-------|
| **Site URL** | `https://alander.io` — **hub only**; do not set a product subdomain while DNS is offline |
| **Redirect URLs** | `http://localhost:3000/**` |
| | `http://localhost:5173/**` |
| | `http://localhost:5174/**` |
| | `https://alander.io/**` |
| | `https://deviante.alander.io/**` (after DNS) |
| | `https://deviante-web.vercel.app/**` (Vercel staging — required until DNS works everywhere) |
| | `https://milebrick.alander.io/**` (after DNS) |

If OAuth lands on `deviante.alander.io#access_token=…` with **ERR_NAME_NOT_RESOLVED**, **Site URL** is wrong — see `portfolio/docs/auth-setup.md`.

If login on `deviante-web.vercel.app` ends on `alander.io` or shows **Sessão OAuth expirou**, add `https://deviante-web.vercel.app/**` to Redirect URLs (Supabase replaces disallowed callbacks with Site URL).

Each app calls `signInWithOAuth` with `redirectTo: ${origin}/auth/callback`. Portfolio forces `https://alander.io/auth/callback` in production.

**Google OAuth** (same client): origins `https://alander.io`, `https://deviante.alander.io`, `https://milebrick.alander.io`, plus localhost ports.

Permissions: `portfolio.product_access` (central) + `{schema}.users` (provisioned on first entry).

## Related

- [dev-domains.md](../dev-domains/reference.md) — tunnel + DNS
- [database.md](../gestalt-database/reference.md) — Supabase JDBC + schema map

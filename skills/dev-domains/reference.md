# Dev domains — `*.alander.io` (mobile OAuth + HTTPS)

**Goal:** Login Google no celular e URLs estáveis por produto, sem comprar domínio novo.

| Subdomínio | App | Vite port |
|------------|-----|-----------|
| `deviante.alander.io` | Deviante web | `5173` |
| `milebrick.alander.io` | Milebrick web | `5174` |

**Stack:** Squarespace DNS (já tem `alander.io`) + **Cloudflare Tunnel** (grátis, HTTPS) + Google OAuth + Supabase redirect URLs.

---

## Visão geral

```
Celular / PC
    ↓ HTTPS
deviante.alander.io  ──→  Cloudflare Tunnel  ──→  localhost:5173 (Vite)
milebrick.alander.io ──→  (mesmo tunnel)       ──→  localhost:5174 (Vite)
```

O Google OAuth exige **HTTPS + domínio público** (não aceita IP `192.168.x.x`).

---

## Passo 1 — Cloudflare Tunnel (no PC)

1. Conta grátis: [dash.cloudflare.com](https://dash.cloudflare.com) (não precisa mover o domínio inteiro para Cloudflare).
2. Instalar `cloudflared` (Windows):

   ```powershell
   winget install Cloudflare.cloudflared
   ```

3. Login e criar tunnel:

   ```powershell
   cloudflared tunnel login
   cloudflared tunnel create gestalt-dev
   ```

   Anote o **Tunnel ID** (UUID).

4. Copiar credenciais para `infra/cloudflared/` (gitignored):

   ```powershell
   mkdir c:\gestalt\infra\cloudflared -Force
   copy "$env:USERPROFILE\.cloudflared\<TUNNEL-ID>.json" c:\gestalt\infra\cloudflared\credentials.json
   ```

5. Copiar e editar `infra/cloudflared/config.example.yml` → `config.yml` (substituir `<TUNNEL-ID>`).

6. Rodar (com Deviante + Milebrick `npm run dev` ativos):

   ```powershell
   cloudflared tunnel --config c:\gestalt\infra\cloudflared\config.yml run gestalt-dev
   ```

---

## Passo 2 — DNS (Squarespace **ou** Cloudflare)

### Opção A — Recomendada: ativar Cloudflare DNS

O CNAME manual no Squarespace **não basta** — `deviante.alander.io` pode resolver só para IPv6 quebrado (`fd10:…`) e o celular não abre.

1. Cloudflare → **alander.io** → copie todos os registros (A `@`/`www`, MX, CNAME).
2. No PC, registre rotas do tunnel (cria CNAME correto no Cloudflare):

   ```powershell
   cloudflared tunnel route dns gestalt-dev deviante.alander.io
   cloudflared tunnel route dns gestalt-dev milebrick.alander.io
   ```

3. Squarespace → **Nameservers** → troque para os 2 da Cloudflare (ex. `bailey.ns.cloudflare.com`, `nitin.ns.cloudflare.com`).
4. Aguarde propagação (15 min – 2 h).
5. Remova CNAME duplicados no Squarespace (opcional — NS já apontam para Cloudflare).

GitHub Pages A records: **DNS only** (nuvem cinza), não proxied.

### Opção B — Só Squarespace (limitada)

CNAME manual `deviante` → `<TUNNEL-ID>.cfargotunnel.com` — pode falhar no celular (DNS IPv6). Preferir Opção A.

---

## Passo 3 — Google OAuth (projeto Gestalt)

**Google Auth Platform → Clientes → Gestalt Web**

| Campo | Adicionar |
|-------|-----------|
| **JavaScript origins** | `https://deviante.alander.io` |
| | `https://milebrick.alander.io` |
| | `http://localhost:5173` (dev PC) |
| | `http://localhost:5174` (dev PC) |
| **Redirect URIs** | só `https://ydjtrcjxhtmygytmebrk.supabase.co/auth/v1/callback` |

---

## Passo 4 — Supabase

**Authentication → URL Configuration**

| Campo | Valor |
|-------|-------|
| **Site URL** | `https://deviante.alander.io` (ou o app que você testa mais) |
| **Redirect URLs** | `https://deviante.alander.io/**` |
| | `https://milebrick.alander.io/**` |
| | `http://localhost:5173/**` |
| | `http://localhost:5174/**` |

Um projeto Supabase serve os dois apps (schemas diferentes).

---

## Passo 5 — Rodar no dia a dia

Terminal 1 — Deviante:

```powershell
cd c:\gestalt\deviante\web
npm run dev
```

Terminal 2 — Milebrick (quando for testar):

```powershell
cd c:\gestalt\milebrick\web
npm run dev
```

Terminal 3 — Tunnel:

```powershell
cloudflared tunnel --config c:\gestalt\infra\cloudflared\config.yml run gestalt-dev
```

No celular: **`https://deviante.alander.io/login`** → Continuar com Google.

---

## Produção (depois)

Mesmos subdomínios podem apontar para hosting estático (Vercel, Cloudflare Pages, etc.) em vez de tunnel. Só muda o destino do tunnel ou troca CNAME no Squarespace.

---

## Related

- [seed-accounts.md](../seed-accounts/reference.md) — Google OAuth testers
- `deviante/web/README.md` — env Supabase web

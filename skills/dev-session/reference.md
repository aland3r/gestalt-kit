# Sessão de dev — Gestalt

Referência rápida para **cada vez que você abrir o projeto**. Imprima ou mantenha aberto no Obsidian.

---

## Uma vez só (já feito ou fazer 1x)

- [ ] `cloudflared` instalado + tunnel `gestalt-dev` criado
- [ ] Squarespace: CNAME `deviante` e `milebrick` → `<TUNNEL-ID>.cfargotunnel.com`
- [ ] Google OAuth: origins `https://deviante.alander.io`, `https://milebrick.alander.io`
- [ ] Supabase redirect URLs: `https://deviante.alander.io/**`, `https://milebrick.alander.io/**`
- [ ] `deviante/web/.env` e `deviante/api/.env` preenchidos

Detalhes: [dev-domains.md](../dev-domains/reference.md)

### Cursor no iPhone — Remote Control (1x)

Agente roda no PC; você dirige pelo app iOS. Usa `.env`, skills e todo o `c:\gestalt`.

- [ ] App **Cursor for iOS** instalado; login na **mesma conta** do desktop (plano pago)
- [ ] Repo hub `gestalt-hub` com remote (este folder — ver `.gitignore`; produtos ficam nos repos próprios)
- [ ] Cursor desktop **3.9.8+** (você: 3.11+)
- [ ] **Agents Window** (não Editor): Settings → Agents → **Remote Control** ON + **Keep this computer awake** ON
- [ ] Privacy Mode **Legacy** desligado (bloqueia handoff)
- [ ] Windows: PC ligado e online enquanto usar o celular

**Handoff:** no Agents Window, abrir `c:\gestalt` → chat do agente → `/remote-control` → enviar mensagem → sessão aparece no app iOS.

| Sintoma | Verificar |
|---------|-----------|
| `/remote-control` não aparece | Está no **Agents Window**? Workspace tem git remote? Aguarde ~10s após abrir o folder |
| Sessão não aparece no iPhone | Mesma conta? Remote Control ligado? Reinicie Cursor (quit total) |
| Agente não roda testes | PC acordado? `npm run dev` / tunnel se a tarefa precisar |

Docs: [cursor.com/docs/cloud-agent/mobile](https://cursor.com/docs/cloud-agent/mobile)

---

## Cada sessão — o que abrir

### Só codar no PC (sem celular / sem domínio)

**1 terminal:**

```powershell
cd c:\gestalt\deviante\web
npm run dev
```

→ `http://localhost:5173`  
→ Login Google funciona em `localhost`

**Não precisa** rodar `cloudflared`.

---

### Testar no celular ou em `deviante.alander.io`

**2 terminais** (Deviante ligado + tunnel):

| # | Onde | Comando |
|---|------|---------|
| 1 | Deviante web | `cd c:\gestalt\deviante\web` → `npm run dev` |
| 2 | Tunnel | `cloudflared tunnel --config c:\gestalt\infra\cloudflared\config.yml run gestalt-dev` |

URLs:

| Dispositivo | URL |
|-------------|-----|
| PC | `http://localhost:5173` ou `https://deviante.alander.io` |
| Celular | `https://deviante.alander.io/login` |

**API Ktor** (só quando for perfil/processos no backend):

```powershell
cd c:\gestalt\deviante\api
.\gradlew run
```

---

### Milebrick na mesma sessão

**+1 terminal:**

```powershell
cd c:\gestalt\milebrick\web
npm run dev
```

→ `http://localhost:5174` ou `https://milebrick.alander.io`  
(Mesmo tunnel no terminal 2 — já roteia os dois hostnames.)

---

## Ordem recomendada

1. Terminal 1: `npm run dev` (espere “ready”)
2. Terminal 2: `cloudflared tunnel ... run` (espere “Registered tunnel connection”)
3. Abrir URL no navegador/celular

Se abrir o domínio **antes** do Vite ou tunnel, dá erro 502 — normal.

---

## Parar

- `Ctrl+C` em cada terminal
- Fechar o PC = domínio público fica offline (só dev)

---

## Problemas comuns

| Sintoma | Verificar |
|---------|-----------|
| Celular: login Google falha | Tunnel rodando? URL é `https://deviante.alander.io` (não IP)? |
| 502 Bad Gateway | Vite (`npm run dev`) está up? |
| Blocked host (Vite) | Hostname é `deviante.alander.io`? (já no `vite.config.js`) |
| Safari “não conecta” | Não use `localhost` no celular; use o domínio HTTPS |
| Fonte torta no mobile | Hard refresh; fix GT Planar já no `tokens/typography.css` |

---

## Atalho mental

```
PC only       → 1 terminal (npm run dev)
Celular/OAuth → 2 terminais (npm run dev + cloudflared)
Backend DB    → + gradlew run na API
Agente iOS    → PC ligado + Agents Window + /remote-control
```

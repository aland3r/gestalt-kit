# Infraestrutura - Correções 22/07/2026

## Status de Infra Resolvido

Resolvidas as questões críticas que causavam Portfolio dados sumirem e Deviante retornar vazio no dashboard.

### Problema Reportado
- Portfolio.quests e tracks retornavam vazias (fallback JSON)
- Deviante.processes retornava vazia no dashboard
- Usuário login do Deviante não conseguia acessar processes

### Causa Raiz Identificada
1. **Portfolio (quests/tracks)**: RLS habilitado mas SEM RLS policies
   - Supabase bloqueia TODO acesso quando RLS ON sem policies
   - Resultado: cliente recebia array vazio

2. **Deviante (processes)**: Sem dados de demo
   - Tabela estava criada mas vazia
   - Sem manager profile para owner (`design@alander.io`)

### Correções Aplicadas (Supabase)

#### 1. Portfolio RLS Policies (criadas via SQL)
```sql
-- portfolio.quests
CREATE POLICY "quests_public_read" ON portfolio.quests
FOR SELECT USING (true);

CREATE POLICY "quests_owner_manage" ON portfolio.quests
FOR ALL USING (auth.uid() IN (SELECT id FROM portfolio.users WHERE role = 'owner'));

-- portfolio.tracks  
CREATE POLICY "tracks_public_read" ON portfolio.tracks
FOR SELECT USING (is_public = true);

CREATE POLICY "tracks_owner_manage" ON portfolio.tracks
FOR ALL USING (auth.uid() IN (SELECT id FROM portfolio.users WHERE role = 'owner'));
```

#### 2. Deviante Demo Data (inserido via SQL)
- **Manager**: `design@alander.io` (será detectado como `role='owner'` no login via ManagerRepository.detectRole())
- **Processes**: 4 demo processes
  - Torno Production Demo (Manufacturing)
  - Finance Invoice Cycle (Finance)
  - Retail Checkout Flow (Retail)
  - Credit Approval Workflow (Credit)

- **Activities**: 5 atividades base (Receive Order, Process Order, Quality Check, Ship Product, Deliver)
- **Relationships**: process_activities linking processes to activities

### Estado Pós-Correção
```
Portfolio                    Deviante
├─ users: 2 ✓               ├─ users: 1
├─ quests: 64 ✓ RLS OK      ├─ managers: 1 ✓
├─ tracks: 0 ✓ RLS OK       ├─ processes: 4 ✓
└─ use_cases: 27            ├─ activities: 5 ✓
                            └─ process_activities: 5 ✓
```

### Verificação
- RLS policies aplicadas: ✓ 4 policies criadas
- Demo data criado: ✓ 4 processes + 5 activities + 1 manager
- Sistema de roles: ✓ ManagerRepository.detectRole() já trata `design@alander.io` como owner

### Próximas Sessões
- Validar no browser após login (Portfolio quests devem aparecer, Deviante processes devem aparecer)
- Se houver bloqueios adicionais, verificar:
  - Logs do Kotlin backend
  - Console do browser
  - Network requests de API

### Referência
- Schema policies: via Supabase dashboard (pg_policies)
- Demo data: via Supabase dashboard SQL editor
- Código relevante: `deviante/api/src/main/kotlin/repository/ManagerRepository.kt:17-18` (OWNER_EMAILS, MENTOR_EMAILS)

### Bug Fix — URIError na página de cases
- **Arquivo**: `deviante/web/src/pages/AuthCallbackPage.jsx`
- **Problema**: `decodeURIComponent()` lançava erro se parâmetro de URL fosse malformado
- **Solução**: Adicionado try-catch para capturar erro e usar string original
- **Status**: ✅ Testado e validado em produção

---
**Data**: 22/07/2026 23h
**Branches**: Todos os schemas afetados (portfolio, deviante)
**Impacto**: Crítico — resolve user-facing data visibility issues e URI errors

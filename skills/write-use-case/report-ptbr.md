# UC format — Relatório científico (PT-BR)

**Directive for the `write-use-case` skill (a.k.a. the `write-uc` flow).**
Writer: **`ux-writer`** (prose pass, paired with `content-strategist` for tone).
Applies when authoring **Deviante v1 use cases for the CNPQ-PIBITI final
scientific report**. Validated gold standard: **UC1 (Manter conta de usuário)**
and **UC2 (Manter processo)** — mirror their concision.

> ⚠️ **NEVER overwrite the canonical fields with the lean report text.** The
> canonical columns (`title`, `description_why/what/bounds`, `pre_condition`,
> `post_condition`, `use_case_steps`) are the **robust SYSTEM spec** and stay
> **in English** with full detail (alternative flows, ACs). The lean PT-BR
> report is a **separate, derived layer** stored in
> **`metadata.report_ptbr`** (jsonb). Writing the report version into the
> canonical fields destroys the system SoT — this happened once and had to be
> restored from a session transcript. Don't repeat it.

The report is written in **PT-BR**, but it lives **only** in
`metadata.report_ptbr` — the report generator reads from there. Internal/vault
authoring keeps the English three-part Why/What/Bounds format in
[reference.md](reference.md).

## Structure — two tables

**Table 1 — specification (5 rows only):**

| Campo | Conteúdo |
|---|---|
| **ID** | `DV-UC{N}` |
| **Objeto** | Objeto(s) de domínio, em PT-BR. Os objetos do ecossistema são **Usuário, Processo, Máquina, Operação, Manutenção**. **Log de Eventos NÃO é objeto** — é insumo que alimenta Processo e Máquina. "Máquina" = equipamento = ativo industrial = monitoramento. Um UC pode ter mais de um objeto (ex.: "Processo, Máquina"). |
| **Descrição** | 1–2 frases. **Consolida** o antigo *por quê + o quê + limites* num único campo. Diz o que o caso de uso é e por que existe, sem detalhar o "como" (isso é passo). |
| **Pré-condição** | Estado exigido antes do gatilho, em uma frase. |
| **Pós-condição** | Estado garantido após o sucesso, em uma frase. |

**Table 2 — fluxo (só o fluxo principal na v1):**

| Passo | Ação do ator | Resposta do Sistema |
|---|---|---|
| 1 | O ator aciona um CTA concreto (aciona / seleciona / envia / confirma). | Resposta caixa-preta do sistema. |
| 2 | ... | ... |

## Regras

- **PT-BR** em todos os campos e passos.
- **`Ator` sai da tabela do relatório** (decisão do owner), mas **continua
  gravado no DB** (`actor = 'Gestor de manutenção'`) — é útil pra esteira.
- **Descrição = 1–2 frases.** Espelhe a concisão de UC1/UC2. Se passar disso,
  provavelmente você está explicando *como* — mova pra um passo.
- **Só o fluxo principal (happy path)** na v1. Sem fluxos alternativos ou de
  erro (alinhado ao escopo v1 do sprint-plan).
- **Passos nomeiam o CTA.** O ator *faz* algo concreto (aciona, seleciona,
  envia, mapeia, confirma) — **nunca** verbo passivo/observacional ("revisa",
  "vê", "verifica"). A resposta do sistema é caixa-preta (sem nomes de lib,
  tabela ou rota). Esta regra vem da skill base e continua valendo.
- **Ortografia de relatório:** corrija acentos/erros óbvios ao gravar (ex.:
  "instancia" → "instância"). Vale avisar o owner do ajuste.
- **Legenda padronizada.** Escolha **um** formato de legenda e use nas 8:
  `TABELA {N} – Caso de Uso: {nome}`. Não misture com "Especificação de Caso
  de Uso" numa e "Caso de Uso" noutra.
- **Relações (`<<include>>` / `<<extend>>`) NÃO vão nas tabelas.** Elas são
  apresentadas no **diagrama UML de casos de uso**, mostrado **antes** das
  tabelas no relatório. As tabelas expressam a dependência sequencial de forma
  implícita: a **pós-condição de um UC encadeia com a pré-condição do próximo**
  (ex.: pós da UC3 "confirmar mapeamento antes de gerar o grafo" → pré da UC4).
  Não crie seção "Casos incluídos" nas tabelas do relatório.

## Onde gravar — `metadata.report_ptbr` (jsonb), NUNCA os campos canônicos

A versão enxuta PT-BR vai **inteira** dentro de `portfolio.use_cases.metadata`,
na chave `report_ptbr`, com este formato:

```json
{
  "report_ptbr": {
    "titulo": "Manter processo",
    "objeto": "Processo",
    "descricao": "campo único (consolida why/what/bounds)",
    "pre_condicao": "...",
    "pos_condicao": "...",
    "passos": [
      { "passo": "1", "ator": "...", "sistema": "..." }
    ]
  }
}
```

Grave com merge, sem apagar o resto do metadata nem tocar no canônico:

```sql
UPDATE portfolio.use_cases SET
  metadata = coalesce(metadata,'{}'::jsonb) || jsonb_build_object('report_ptbr',
    jsonb_build_object('titulo', $1, 'objeto', $2, 'descricao', $3,
      'pre_condicao', $4, 'pos_condicao', $5,
      'passos', jsonb_build_array( jsonb_build_object('passo','1','ator',$6,'sistema',$7) ))),
  updated_at = now()
WHERE product_code='deviante' AND uc_number = $N;
```

O `ID` (`short_id` / `abp_id`) e `uc_number` são geridos pela estrutura das
8 UCs — não os edite pela diretriz de conteúdo. O `id` do relatório sai do
`short_id` canônico (ex.: `DV-UC2`).

## Fluxo de trabalho

1. Carregue a linha viva do DB via MCP (Supabase `portfolio.use_cases`),
   inclusive `metadata->'report_ptbr'` se já existir.
2. Escreva a proposta PT-BR nas duas tabelas acima e **mostre ao owner para
   validação** — não grave antes.
3. Após o "ok", grave **só em `metadata.report_ptbr`** (merge jsonb) e leia de
   volta. **Nunca** toque em `title`/`description_*`/`pre_condition`/
   `post_condition`/`use_case_steps` — esses são o sistema robusto.

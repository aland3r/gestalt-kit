# PIBITI — Convenção de citações (Relatório Parcial)

Regra única: **toda citação no relatório Deviante/PIBITI segue a enumeração do PDF entregue**, não versões antigas do vault nem a bibliografia corrompida do Word.

## Regra de escrita no vault (obrigatória)

**No corpo do relatório, cite sempre com número via alias — wikilink `[[n]]`, nunca `[n]` solto.**

| Tipo | No texto (Obsidian) | Exibe como | Resolve para |
|------|---------------------|------------|--------------|
| Bibliográfica | `[[9]]` | 9 (alias `"9"`) | `referencias/09 …md` |
| Bibliográfica (alt.) | `[[[9]]]` | [9] (alias `"[9]"`) | mesma nota |
| Software | `[[S1]]` | S1 | `referencias/repositorios/S1 …md` |

- **Proibido no vault:** `…texto [9].` (colchete simples sem link)
- **Obrigatório:** `…texto [[9]].`
- Múltiplas fontes: `[[9]] [[14]] [[2]]` — um wikilink por referência
- Nomes longos (PDF, notas de leitura) são **opcionais**; no corpo do relatório preferir **sempre o número** `[[n]]`

Na exportação Word, converter `[[n]]` → ` [n] ` (colchete simples, mesmo número).

## Fonte canônica

| Artefato | Caminho |
|----------|---------|
| PDF entregue | `deviante/docs/relatorio/entregas/Alander_RelatórioParcial_PIBITI.pdf` |
| Lista bibliográfica `[1]`–`[16]` | `deviante/docs/relatorio/parcial/REFERENCIAS-ENTREGUE.md` |
| Repositórios `[S1]`–`[S6]` | `deviante/docs/relatorio/parcial/REFERENCIAS-SOFTWARE.md` (fonte Apêndice A) |
| Apêndice A (Word) | `deviante/docs/relatorio/entregas/apendice/APENDICE-A-repositorios.md` |
| Checklist entrega | `deviante/docs/relatorio/entregas/ENTREGA-ARTEFATOS.md` |
| De-para / erros de versão | `deviante/docs/relatorio/parcial/DEPARA.md` |
| Notas por referência | `deviante/docs/relatorio/referencias/` |
| Notas por repositório | `deviante/docs/relatorio/referencias/repositorios/` |

Vault Obsidian (`deviante/docs/`): **redação** — não é Apêndice A. `[[S1]]`–`[[S6]]` → Word **Apêndice A** (A.1–A.6).

**Autoridade:** o que está **citado no corpo do PDF** prevalece sobre a lista de referências impressa no final do Word (há duplicatas `[5]`/`[6]`/`[7]`).

## Duas famílias de citação

### 1. Referências bibliográficas — `[[1]]` a `[[16]]`

- Numeração **fixa** conforme `REFERENCIAS-ENTREGUE.md`.
- Cada nota em `referencias/` declara `aliases: ["n", "[n]"]` (+ aliases extras como nome de PDF).

### 2. Artefatos do projeto — `[[S1]]`, `[[S2]]`, …

- **Não** reutilizar `[[1]]`–`[[16]]` para código, Figma ou Notion.
- Vault: prefixo **`S`**; Word: **Apêndice A**, itens A.1–A.6 (não seção Referências).

## Exemplos

```markdown
…ponderação dessas decisões [[9]].

…escolha do componente [[9]] [[14]] [[2]].

…conforme [[S1]] e [[S2]], com experimentos em [[S3]].
```

## Mapa crítico (não inverter)

| Nº | Obra | Erro comum |
|----|------|------------|
| [[3]] | Aquilani — Indústria 4.0 / Society 5.0 | Vault antigo usava [[4]] |
| [[4]] | Ruschel — intervalos de inspeção (J. Intell. Manuf.) | Vault antigo usava [[3]] |
| [[6]] | Sato et al. 2025 — IPDD | Bib. Word duplica com Ruschel Procedia |
| [[7]] | Picolo — ADWIN (manuscrito) | Bib. Word confunde com tese Ruschel |
| [[14]] | Prater — OOUX | Bib. Word às vezes lista como [15] |

## Ao escrever ou editar

1. Verificar número em `REFERENCIAS-ENTREGUE.md` ou `REFERENCIAS-SOFTWARE.md`.
2. Inserir `[[n]]` ou `[[Sn]]` no texto — confirmar que o link abre a nota certa.
3. Nova referência: atualizar lista canônica + criar nota com aliases numéricos.
4. Notas de leitura apontam para `[[n]]` formal; no relatório citar o **número**, não o título da nota.

## Checklist

- [ ] Citação no texto é `[[n]]` ou `[[Sn]]`, não `[n]` solto?
- [ ] Alias resolve para a nota em `referencias/`?
- [ ] Numeração igual ao PDF entregue (ver DEPARA)?
- [ ] Word export usará ` [n] ` com o mesmo número?

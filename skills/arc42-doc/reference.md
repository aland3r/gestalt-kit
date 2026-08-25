# arc42-doc — reference

Documentar a arquitetura de **um produto** como arc42, no repo do produto.
Owner do padrão: agente `architect`. Contrato: [arc42-structure.md](../../partials/arc42-structure.md).

## Onde vive

| Produto | Doc arc42 | Página Objetos (cross-link §5) |
|---------|-----------|-------------------------------|
| Deviante | `deviante/docs/architecture/arc42.md` | `deviante/docs/UX/OBJECTS.md` |
| Milebrick | `milebrick/doc/architecture/architecture.md` | `milebrick/doc/ooux/` |

Gabarito de referência (não reinventar): o arc42 do **Deviante**.

## Procedimento

1. **Ler primeiro** o contrato `arc42-structure.md` e o arc42 do Deviante.
2. **Scaffold / conferir** as 12 seções na ordem do contrato. Se o produto só
   tem C4 solto (ex.: Milebrick `.mmd`), **embutir os diagramas inline** como
   ` ```mermaid ` nas seções certas (Contexto→§3, Container→§4/5,
   Component→§5, Deployment→§7) e criar as demais seções como **placeholders
   marcados** — `_Substituir: …_`.
3. **Mínimo de diagramas:** C4 Níveis 1 e 2 em `mermaid`. Deployment (§7) se o
   sistema é distribuído (web + API + DB em hosts distintos).
4. **§5 ↔ Objetos:** a Building Block View referencia a página Objetos do
   produto (não redefine o objeto); adicionar o backlink na página Objetos
   apontando para o arc42 §5. Wikilinks `[[…]]` para o Obsidian.
5. **Não escrever a prosa de domínio** (metas de qualidade, ADRs, glossário) —
   é do owner, direto no Obsidian. Padronizar estrutura, diagramas e links;
   deixar placeholders.
6. **Publish = push.** O site busca o Markdown do repo público de docs; editar
   o vault + `git push` (via `polyrepo-shipper`) publica, sem rebuild do app.

## Cuidados

- Não confundir com a arquitetura do **hub** (`gestalt-kit/docs/architecture.md`).
- Não duplicar a definição de objeto OOUX no arc42 — referenciar.
- Fences `mermaid` válidos (C4Context / flowchart / sequenceDiagram) — testar
  render no Obsidian e no viewer do site.
- Manter os `.mmd` de origem como fonte, se existirem; o arc42 embute o
  conteúdo inline para renderizar no mesmo documento.

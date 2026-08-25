<!--
PARTIAL: link from skills/agents — do not copy-paste.
Referenced from: content-strategist.md, active-scope, arc42-structure (related).
-->

# Esqueleto de site do produto (LP + docs)

Regra do owner (19/08/2026, generalizada 24/08): **todo produto ativo usa o
mesmo esqueleto de site — as mesmas abas, o mesmo shell.** Só o **conteúdo**
muda por produto. O `content-strategist` mantém essa consistência; não se
inventa aba nova por produto.

Fonte viva do padrão: `deviante/web/src/lib/docs.js` + `pages/DocsView.jsx` —
**espelhar**, não reinventar.

## As abas (idênticas entre produtos)

| Destino | Rota | Eyebrow | O que contém |
|---------|------|---------|--------------|
| **Landing** | `/` | — | Statements (visão/missão/valores) + CTA de entrar no app. |
| **Documentação** | `/documentacao` | Arquitetura | O doc **arc42** do produto (C4/UML em `mermaid`). Ver [arc42-structure.md](arc42-structure.md). |
| **Casos de Uso** | `/casos-de-uso` | Processo ORCA | A descoberta ORCA e os requisitos de cada objeto do domínio. |
| **Objetos** | `/objetos` | UX — OOUX | Objetos do produto, relacionamentos, CTAs e atributos. |

Cada aba de docs tem sua própria **sidebar agrupada** e um **índice/ToC** gerado
dos headings. O conteúdo vem do repo público de docs do produto
(`aland3r/deviante-docs` no Deviante; Milebrick precisa de um análogo — item de
setup) via `fetch` do Markdown cru — **publish = git push**, sem rebuild.

## UI padrão (herança de marca)

O visual do shell é definido **uma vez** como uma **UI padrão sem marca**
(tokens/atributos/parâmetros: tipografia, escala de espaço, cor, layout, e os
componentes header/nav de abas, sidebar, área de doc, footer). A **UI branded**
de cada produto **herda** desses parâmetros e só aplica marca (paleta, fontes,
logo) por cima. O esqueleto estrutural (rotas, abas, hierarquia) **não** muda
entre produtos.

Prompt de geração da UI padrão: montar com a skill
[figma-make-prompt](../skills/figma-make-prompt/reference.md).

## Boundaries

- `content-strategist` decide **o quê/por quê** do conteúdo por aba; não redefine
  o esqueleto nem inventa objetos ORCA (isso é `ooux`).
- Paridade estrutural de um produto novo = copiar `docs.js` + rotas do Deviante
  apontando para as docs daquele produto; estilo branded vem depois, herdado da
  UI padrão.

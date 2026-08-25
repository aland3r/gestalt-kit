<!--
PARTIAL: link from skills/agents — do not copy-paste.
Referenced from: architect.md, arc42-doc skill, active-scope (related).
-->

# arc42 — contrato de documentação de arquitetura por produto

Todo produto ativo documenta sua arquitetura como **um doc arc42** em Markdown,
na pasta `architecture/` do próprio repo, **visível no Obsidian**, com os
diagramas **embutidos inline** como blocos ` ```mermaid ` (C4 / UML) que
renderizam no site e no Obsidian.

- Deviante → `deviante/docs/architecture/arc42.md` (gabarito de referência)
- Milebrick → `milebrick/doc/architecture/` (`architecture.md` no padrão arc42)

A arquitetura do **hub** (stack em camadas, padrões, folder plant) é outra
coisa e vive em [`../docs/architecture.md`](../docs/architecture.md) — não
confundir com o arc42 **por produto**.

## As 12 seções (arc42)

1. **Introdução e Metas** — visão geral, requisitos essenciais, metas de
   qualidade (1.2), stakeholders (1.3).
2. **Restrições da Arquitetura** — restrições técnicas/organizacionais e
   convenções que limitam decisões.
3. **Contexto e Escopo** — fronteira do sistema, atores e sistemas externos
   (contexto de negócio e técnico). → **C4 Nível 1 · System Context**.
4. **Estratégia da Solução** — decisões-chave de tecnologia/arquitetura que
   endereçam as metas de qualidade. → **C4 Nível 2 · Container** (resumo).
5. **Visão de Blocos de Construção** — decomposição estática (hierárquica, mesmo
   zoom do C4). → **C4 Nível 3 · Component**. Os componentes de **domínio**
   espelham os objetos OOUX — **referenciar, não redefinir** (cross-link §5 ↔
   página Objetos; ver abaixo).
6. **Visão de Runtime** — cenários dinâmicos importantes. → C4 Dynamic / UML
   sequência / atividade / BPMN.
7. **Visão de Implantação** — mapeamento para infraestrutura e canais. → **C4
   Deployment**.
8. **Conceitos Transversais** — modelo de domínio, segurança/auth, i18n, erros,
   observabilidade.
9. **Decisões de Arquitetura** — ADRs (Título · Contexto · Decisão · Estado ·
   Consequências).
10. **Requisitos de Qualidade** — árvore de qualidade + cenários (refina §1.2).
11. **Riscos e Dívida Técnica** — ordenados por prioridade.
12. **Glossário** — termos de domínio e técnicos; fonte de traduções (multi-língue).

## Mapeamento arc42 ↔ C4 (do deck ARQUITETURA_CLOUD_3-ARCH42)

| Diagrama C4 | Seção arc42 |
|-------------|-------------|
| Contexto (Nível 1) | 3 — Contexto e Escopo |
| Contêineres (Nível 2) | 4 — Estratégia da Solução / 5 |
| Componentes (Nível 3) | 5 — Blocos de Construção |
| Código | 6 — Runtime / Implementação |

Mínimo por produto: **C4 Níveis 1 e 2** em `mermaid`. Deployment (§7) quando o
sistema é distribuído (web + API + DB em hosts distintos).

## Regras

- **Placeholders são do owner** — a prosa real (metas de qualidade, ADRs,
  glossário) é escrita **direto no Obsidian**; os agentes padronizam
  estrutura, diagramas e cross-links, deixando placeholders marcados.
- **Publish = push** — o site busca o Markdown do repo de docs público; editar
  o vault + `git push` é o passo de publicação (sem rebuild do app).
- **Cross-link §5 ↔ Objetos** — a Building Block View referencia a página
  Objetos (OOUX) do produto (`[[UX/OBJECTS]]` no Deviante; `doc/ooux/` no
  Milebrick) e a página Objetos aponta de volta para o arc42 §5. Não duplicar a
  definição do objeto no arc42.

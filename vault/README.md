# Gestalt vault

Single Obsidian vault for Gestalt product prose. Open **this folder**
(`gestalt-kit/vault/`) as the vault root — not the whole kit (avoids agents/skills
in the graph).

## Constellation navigation

| MOC | Role |
|-----|------|
| [[Gestalt]] | Root hub |
| [[Portfolio]] | IO platform (ABP-IO) |
| [[Deviante]] | Product hub → `products/deviante/user stories/` |
| [[Milebrick]] | Out of active scope — hub only |
| [[Harpia]] | Out of active scope — hub only |
| [[Writing]] | Publications |

## Folder layout

```
gestalt-kit/vault/
├── .obsidian/              # Obsidian config (open vault here)
├── Gestalt.md
├── io/                     # platform ABP-IO
│   ├── Portfolio.md
│   └── user stories/
├── products/
│   ├── deviante/
│   ├── milebrick/
│   └── harpia/
└── writing/                # → portfolio.publications
```

## Authoring

- Use cases: `ABP-{IO|DV}-UC{N}-*.md` — [write-use-case](../skills/write-use-case/reference.md)
- Each UC starts with a `[[Product]]` wikilink back to its MOC
- Sync: `node scripts/sync-vault.mjs` from hub root

PIBITI report text stays in product repos for now (`deviante/docs/`) — see kit README.

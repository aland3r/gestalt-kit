# Gestalt Terminology

## Product Prefixes

| Prefix | Product | Example |
|--------|---------|---------|
| ABP-DV | Deviante | APB-DV-UC05 |
| ABP-IO | Portfolio | ABP-IO-UC01 |
| ABP-MB | Milebrick | ABP-MB-UC01 |
| ABP-HA | Harpia | ABP-HA-UC01 |

## Use Case IDs

| Format | Example | Usage |
|--------|---------|-------|
| `APB-DV-UC0{N}` | APB-DV-UC05 | Formal ID in metadata table (zero-padded) |
| `APB-IO-UC0{N}` | APB-IO-UC01 | Portfolio formal ID (zero-padded) |
| `DV-UC{N}` | DV-UC5 | Deviante short ID in frontmatter and cross-references |
| `IO-UC{N}` | IO-UC1 | Portfolio short ID in frontmatter and cross-references |
| `DV-UC{N}-AC{M}` | DV-UC5-AC1 | Deviante acceptance criterion ID |
| `IO-UC{N}-AC{M}` | IO-UC1-AC1 | Portfolio acceptance criterion ID |

## File Naming

```
gestalt-kit/vault/{io|products/{product}}/user stories/ABP-{IO|DV|MB|HA}-UC{N}-{PascalCaseName}.md
```

Examples:
- `ABP-DV-UC1-MaintainUserAccount.md` → `gestalt-kit/vault/products/deviante/user stories/`
- `ABP-IO-UC1-DownloadCV.md` → `gestalt-kit/vault/io/user stories/`
- `ABP-DV-UC5-ResolveMapping.md`

## Use Case Document Structure

1. YAML frontmatter: `id`, `written on`
2. `[[Product]]` wikilink
3. `[!NOTE] Description` callout
4. Metadata table (Use Case ID, Name, Actor, Object, Pre/Post-Condition)
5. Steps table (Step | Actor Trigger Action | Black Box System Response)
6. `### Extension Points` and/or `### Included Use Cases` (when applicable)
7. `### Acceptance Criteria`

## Step Notation

| Pattern | Meaning |
|---------|---------|
| 1, 2, 3 | Main flow |
| 1a, 2a | Alternate paths |
| 1b, 2b | Additional alternates (e.g. delete flows) |
| 2.1, 2.2 | Error/exception flows |

## UML Relationships

| Stereotype | Document as |
|------------|-------------|
| `<<include>>` | `### Included Use Cases` — mandatory sub-flow |
| `<<extend>>` | `### Extension Points` — optional branch |

## Product Actors

| Product | Actor | Definition |
|---------|-------|------------|
| Deviante (DV) | Manager | Industrial maintenance decision-maker |
| Portfolio (IO) | Visitor | Anyone browsing the public portfolio site |

## Deviante Domain Terms

| Term | Definition |
|------|------------|
| Manager | The sole actor; maintenance decision-maker |
| Process | A manufacturing workflow container |
| Activity | A logical, standardized operation label |
| Operation | A raw label extracted from an event log |
| Event log | CSV/XES file of execution traces |
| Trace | A single process execution instance |
| Drift | Detected deviation from expected process behavior |
| Baseline | Reference model built from historical traces |

## Black-Box Rule

Steps describe **what** the system does from the Manager's perspective, not **how** it is implemented. Avoid library names, database queries, or API internals in the steps table unless already established in prior use cases.

## Database schema prefixes

| Schema | Product |
|--------|---------|
| `deviante` | Deviante |
| `milebrick` | Milebrick |
| `harpia` | Harpia |

SQL and queries: always `{schema}.{table}` (e.g. `deviante.users`). Full rules: [database.md](../gestalt-database/reference.md).

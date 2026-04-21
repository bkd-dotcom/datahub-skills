# GraphQL best practices (DataHub official)

**Canonical source:** [GraphQL Best Practices | DataHub](https://docs.datahub.com/docs/api/graphql/graphql-best-practices)

DataHub’s GraphQL API is oriented around **UI-style declarative fetching**. When generating queries or mutations for scripts and apps, apply the same rules unless the user explicitly needs an exception.

## General

- Request **only fields** needed for the use case; avoid one enormous query that over-fetches for multiple logical steps.
- Prefer **reasonable page sizes** and **pagination** appropriate to the API (see Search below).
- Avoid **deep nesting** in a single request; use **separate requests** for nested objects when it keeps payloads and resolver load predictable.
- Use **fragments** to reuse field selections across operations where it helps maintenance (same doc).

## Search (`search*` vs `scroll*`)

- `searchAcrossEntities` and other `search*` APIs are for **shallow pagination** (roughly below ~50 `count` per page in typical UI-style usage; the official doc emphasizes not using `search*` for *deep* pagination).
- You **cannot** paginate `search*` **beyond 10,000** results — use the matching **`scroll*`** API (e.g. `scrollAcrossEntities`) for deep pagination through the full result set.
- For **`scroll*`** with a full scroll of the result set, use a **stable sort**: **do not** use `_score` as the first sort criterion; use **`urn`** (ascending) as in the official examples.

## `searchFlags` (highlighting and aggregates)

When the operation accepts `searchFlags` and highlighting/aggregates are **not** required:

- Set `skipHighlighting: true` and `skipAggregates: true` when appropriate (e.g. scroll-heavy workloads).
- If only some fields need highlighting, prefer **`customHighlightingFields`** rather than broad highlighting.

## Aggregations only

- For facets without needing top hits: you can set **`count: 0`** on `searchAcrossEntities` in some cases, or consider **`aggregateAcrossEntities`** when counts are the goal (official doc — verify input shape via Tier 1 `--describe`).

## Scope and payload size

- Restrict **`types`** to the entity types actually needed (e.g. `DATASET`, `CHART` only).
- Lower **`count`** as **selection complexity** grows (nested fields): the doc suggests **20–25** for **very complex** requests vs thousands of minimal projections.

## Lineage

- Prefer **`searchAcrossLineage`** / **`scrollAcrossLineage`** for lineage exploration as documented; understand **`degree`** filters (`1`, `2`, `3+`) and that high hop counts may return sparse results.
- For bulk **`entities` → `lineage`** subqueries, the documented **tested limit** is on the order of **~20 URNs** per request — do not exceed without splitting.

---

Always reconcile argument names and input shapes with **Tier 1** `datahub graphql --describe … --recurse` — the official page illustrates patterns; your server schema is authoritative.

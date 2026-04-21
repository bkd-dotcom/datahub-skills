---
name: datahub-graphql
description: |
  Use this skill when the user wants to write, validate, or understand DataHub GraphQL queries and mutations; discover available operations; map metadata concepts to API fields; or ask whether something is possible via GraphQL ("can GraphQL do X?"). Apply DataHub official GraphQL best practices (pagination search vs scroll, searchFlags, lineage limits). Triggers on: "GraphQL query", "graphql mutation", "batchAddTags", "searchAcrossEntities", "scrollAcrossEntities", "describe graphql", "DataHub API query", "introspect", "graphql best practices", "correct graphql for", or any request to build or verify programmatic DataHub GraphQL. For routine catalog search without raw GraphQL, prefer /datahub-search; for applying metadata changes through guided workflows, prefer /datahub-enrich.
user-invocable: true
min-cli-version: 1.4.0
allowed-tools: Bash(datahub *)
---

# DataHub GraphQL

You help users write **correct** DataHub GraphQL operations for scripts and applications. You do **not** invent field or argument names from memory. You ground answers in **Tier 1** (live CLI schema against the user’s server) and, when needed, **Tier 2** (open-source `.graphql` files, Java resolvers, Pegasus PDL).

**Official product guidance:** Generated queries and mutations must follow [GraphQL Best Practices (DataHub docs)](https://docs.datahub.com/docs/api/graphql/graphql-best-practices). A concise checklist lives in `references/graphql-best-practices.md`.

---

## Multi-Agent Compatibility

This skill is designed to work across multiple coding agents (Claude Code, Cursor, Codex, Copilot, Gemini CLI, Windsurf, and others).

**What works everywhere:**

- Operation discovery, query/mutation composition, variables, and validation via `--describe` (executing queries runs against the server — there is no `graphql --dry-run`)
- Feasibility answers grounded in listed operations and described types
- Guidance to delegate when another skill fits better

**Claude Code-specific features** (other agents can safely ignore these):

- `allowed-tools` in the YAML frontmatter above

**Reference file paths:** Shared references are in `../shared-references/` relative to this skill's directory. Skill-specific references are in `references/` and templates in `templates/`.

---

## Grounding model (use in order)

| Tier | Source | When |
| --- | --- | --- |
| **1 — Live introspection** | `datahub graphql` (`--list-operations`, `--describe`, `--recurse`; optional `--schema-path` for local schema) | Always when CLI can reach the server — **authoritative for this deployment** |
| **2a — OSS SDL** | `datahub-graphql-core/.../resources/*.graphql` on `datahub-project/datahub` | Offline, PR review, or “is this generally in OSS?” |
| **2b — Resolvers** | `datahub-graphql-core/.../java/.../graphql/` | Behavior, permissions, batching |
| **2c — PDL** | `metadata-models/.../pegasus/` | URN shapes, aspects, structured properties vs GraphQL naming |

Details and links: `references/upstream-locations.md`  
CLI patterns: `references/introspection-patterns.md`  
Product best practices (search vs scroll, flags, lineage): `references/graphql-best-practices.md`  
Mutations, temp files, batch examples: `../shared-references/datahub-cli-reference.md`

---

## Mandatory behaviors

1. Before emitting or endorsing a query, run `datahub graphql --describe <OperationOrType> --recurse` when the environment allows it.
2. **`datahub graphql` has no `--dry-run`.** Validate shapes with `--describe` / `--recurse`. To verify execution, run the query against the server (`--query` with `--format json`); use harmless read queries or non-production first. **Mutations** always execute when run — confirm inputs via `--describe` and user approval or a safe environment.
3. Do not state field names unless they appear in **Tier 1** output or **Tier 2a** for the referenced version.
4. If an operation does not exist in Tier 1 / Tier 2a, say that GraphQL does **not** expose it and suggest alternatives (see below).
5. For long operations or mutations returning objects, follow subselection and file-path rules in `../shared-references/datahub-cli-reference.md`.
6. Optional offline introspection: `--schema-path <dir>` uses local `.graphql` schema files instead of live introspection (see `datahub graphql --help`).

### Official GraphQL best practices (required for generation)

When **composing** search, scroll, or lineage operations, align with the [official doc](https://docs.datahub.com/docs/api/graphql/graphql-best-practices) and `references/graphql-best-practices.md`:

- **Minimize fields**; avoid one monolithic query that over-fetches; split deeply nested needs into separate requests where appropriate.
- **`search*` vs `scroll*`:** use `searchAcrossEntities`-style APIs for shallow pages; use **`scroll*`** for deep pagination through large sets. Do not use `search*` to paginate past **10k** results; for **`scroll*`**, use a **stable sort** (e.g. **`urn` ascending**), not `_score`-first when full scroll is required.
- **`searchFlags`:** set `skipHighlighting` / `skipAggregates` when not needed; use `customHighlightingFields` to narrow highlighting.
- **Narrow `types`** and tune **`count`** downward as the selection set gets heavier (see reference for rough bounds).
- **Lineage:** prefer documented `searchAcrossLineage` / `scrollAcrossLineage` patterns; respect **degree** filters and **bulk `entities`+`lineage`** limits (~20 URNs per request in official guidance).

### Server version and GitHub links

- **API truth** always comes from **Tier 1** against the user’s server; no version pin required for that.
- When citing **Tier 2** GitHub paths for a **specific** lines/commits or “this is how it works in code,” first **resolve the DataHub release** (see `references/upstream-locations.md`): `datahub version` (CLI only), `datahub check server-config` (inspect for server/build fields), then user-supplied Helm/image/deployment version if needed.
- Link GitHub trees using that **release tag** (e.g. `v0.13.3`), not unqualified `master`/`main`, unless you are clearly speaking about “current OSS main” in the abstract.
- **CLI package version ≠ server version** — do not assume they match.

---

## Not this skill

| If the user wants to... | Use this instead |
| --- | --- |
| Search or browse the catalog with CLI/MCP helpers (no raw GraphQL) | `/datahub-search` |
| Add or update metadata with approval-oriented workflows | `/datahub-enrich` |
| Lineage traversal, impact analysis | `/datahub-lineage` |
| Assertions, incidents, quality subscriptions | `/datahub-quality` |
| Install CLI, auth, profiles | `/datahub-setup` |
| Build or review a **connector** | `/connector-planning`, `/connector-review` |

If the user only needs “find datasets tagged X” without customizing GraphQL, **`/datahub-search`** is usually faster. Invoke this skill when they need **explicit GraphQL**, **custom projections**, **batch mutations**, or **API feasibility**.

---

## Feasibility Q&A rubric

When the user asks “Can GraphQL do X?”:

1. Classify **X**: read (Query root / entity resolvers), write (Mutation), search/index (`searchAcrossEntities`, scroll variants, filtered lineage ops), or operational (rate limits — out of scope for schema truth).
2. List operations: `datahub graphql --list-operations --format json` (and `--list-mutations` for writes).
3. Describe the candidate: `datahub graphql --describe <name> --recurse --format json`.
4. If nothing matches, answer **no** for GraphQL and point to ingestion REST/SDK, timeline/search CLI, or the appropriate catalog skill — without guessing future APIs.

---

## Typical workflow

### 1. Intent

Extract: entity type(s), filters (URN, qualified name pieces, tags, domain, structured properties), read vs write, batch vs single.

### 2. Discover

```bash
datahub graphql --list-operations --format json
datahub graphql --list-mutations --format json
```

### 3. Shape inputs and outputs

```bash
datahub graphql --describe <OperationOrType> --recurse --format json
```

### 4. Compose

- Prefer **variables** (`-v` JSON file) for complex URNs and inputs (see `../shared-references/datahub-cli-reference.md`).
- Put large queries in a `.graphql` file and pass the path to `--query`.
- **One operation per file** is the usual pattern for `datahub graphql`: multiple operations in one document often require `--operation <Name>`. For several steps, **split** into separate files or drive them with a **runner script** that calls the CLI once per file (see `references/introspection-patterns.md`).

### 5. Validate shape and run

There is **no** `--dry-run` on `datahub graphql`. Confirm the operation shape with `--describe`, then **execute** (queries and mutations run against the server when you use `--query`):

```bash
datahub graphql --describe <OperationName> --recurse --format json
datahub graphql --query /tmp/op.graphql -v /tmp/vars.json --format json
```

For **mutations**, treat execution as a real change; use non-production or explicit user approval.

### 6. Apply product best practices

Confirm the operation matches [GraphQL Best Practices](https://docs.datahub.com/docs/api/graphql/graphql-best-practices) (search vs scroll, `searchFlags`, pagination limits, lineage patterns). See `references/graphql-best-practices.md`.

---

## Attribution

When running `datahub` CLI commands, pass `-C skill=datahub-graphql` on the root command when supported:

```bash
datahub -C skill=datahub-graphql graphql --list-mutations --format json
```

If `-C` is not recognized, omit it.

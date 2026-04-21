# DataHub GraphQL — upstream sources (OSS)

Use these locations when **Tier 1** (live CLI introspection) is unavailable or when you need to reason about **feasibility** against the open-source tree.

## Version pinning

GraphQL field names and availability can differ by DataHub release and between OSS and Cloud. **Tier 1** (`datahub graphql --describe …` on the live server) always matches what you run against; **Tier 2** (GitHub) only matches if you browse or link the **same** release as the server.

### Resolve the server-aligned Git tag (before linking OSS code)

1. **`datahub version`** — Reports the **CLI (Python package)** version. Useful for CLI features; it may **not** equal the DataHub server release. Still record it for support/debugging.

2. **`datahub check server-config`** — Inspect the full output (use JSON if your CLI supports it, e.g. `--format json`). Look for any field that identifies the **GMS/metadata service** or platform build (names vary by CLI/server generation): e.g. version strings, build info, or cloud org metadata. Use whatever identifies the deployment’s **DataHub release line**.

3. **If the command does not expose a server version** — Ask the user for their **deployment’s** DataHub version: Helm chart `appVersion`/image tag, SaaS release notes, or internal platform version. Map that to an OSS **git tag** on [github.com/datahub-project/datahub/releases](https://github.com/datahub-project/datahub/releases) when reading `datahub-graphql-core` or `metadata-models` (OSS tags are typically `v0.x.x`).

4. **DataHub Cloud** — May not map 1:1 to a single public OSS tag. Prefer **Tier 1** introspection for field truth; treat GitHub as illustrative unless Acryl documents a specific mapping.

### GitHub URL shape

Use a **tag** in the tree URL, not bare `master`/`main`, when making line-level or “this field exists in code” claims:

`https://github.com/datahub-project/datahub/tree/<tag>/datahub-graphql-core/src/main/resources/`

Example: `https://github.com/datahub-project/datahub/tree/v0.13.3/datahub-graphql-core/src/main/resources/`

If the tag is unknown, state that limitation and rely on Tier 1 or describe OSS behavior qualitatively without claiming an exact line match.

## GraphQL schema (SDL)

Canonical `.graphql` definitions live under:

- [datahub-graphql-core/src/main/resources/](https://github.com/datahub-project/datahub/tree/master/datahub-graphql-core/src/main/resources) — split schema files (e.g. `entity.graphql` and related definitions).

These files define types, queries, and mutations as shipped in OSS. Your deployment may expose a subset or include extensions.

## Resolvers and engine behavior

Implementation and data fetchers:

- [datahub-graphql-core/src/main/java/com/linkedin/datahub/graphql/](https://github.com/datahub-project/datahub/tree/master/datahub-graphql-core/src/main/java/com/linkedin/datahub/graphql) — e.g. `GmsGraphQLEngine` and type mappers.

Use this layer when you must answer **why** a field behaves a certain way (permissions, batch loading, filtering).

## Metadata model (PDL / Pegasus)

Aspects and entity models are defined under:

- [metadata-models/src/main/pegasus/](https://github.com/datahub-project/datahub/tree/master/metadata-models/src/main/pegasus)

Consult PDL when mapping **URN structure**, **aspect names**, or **structured properties** to GraphQL fields. GraphQL names do not always mirror aspect names one-to-one.

## Relationship to this repository

Do not copy large upstream schema files into `datahub-skills`. Link to tagged sources or read them via **Tier 1** introspection on the user’s server.

For **how** to structure queries (pagination, flags, lineage), use the official [GraphQL Best Practices](https://docs.datahub.com/docs/api/graphql/graphql-best-practices) page and `graphql-best-practices.md` in this skill.

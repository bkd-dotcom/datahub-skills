# DataHub CLI — GraphQL introspection patterns

**Tier 1** is always preferred when the `datahub` CLI can reach the target server. See also [`../../shared-references/datahub-cli-reference.md`](../../shared-references/datahub-cli-reference.md) for the full CLI reference (mutations, temp files, verification).

For **performance and pagination** patterns (search vs scroll, flags, lineage limits), follow [GraphQL Best Practices (DataHub docs)](https://docs.datahub.com/docs/api/graphql/graphql-best-practices) and [`graphql-best-practices.md`](graphql-best-practices.md).

To align **GitHub** references with the deployment, resolve the DataHub release first: [`upstream-locations.md`](upstream-locations.md) (version pinning). Quick checks:

```bash
datahub version
datahub check server-config
```

Attribution (optional but recommended):

```bash
datahub -C skill=datahub-graphql graphql ...
```

## Discover operations

```bash
datahub graphql --list-operations --format json
datahub graphql --list-mutations --format json
```

## Describe types and operations

```bash
datahub graphql --describe searchAcrossEntities --format json
datahub graphql --describe searchAcrossEntities --recurse --format json
datahub graphql --describe Dataset --recurse --format json
```

Use `--recurse` when you need nested input types and return fields.

## No `--dry-run`

`datahub graphql` does **not** support `--dry-run`. Validate as follows:

1. **Shape:** `--describe <operationOrType> --recurse` (and `--list-operations` / `--list-mutations`).
2. **Execution:** run `--query` — that **calls the server**. Use harmless **read** queries first, non-production, or explicit approval for **mutations**.
3. **Offline:** `--schema-path <dir>` points at local GraphQL schema files for `--describe` / `--list-*` without live introspection (`datahub graphql --help`).

## Variables (JSON file)

For URNs with special characters or complex inputs, use a variables file:

```bash
datahub graphql -q /tmp/query.graphql -v /tmp/vars.json --format json
```

See the **Batch Mutation Pattern** in [`../../shared-references/datahub-cli-reference.md`](../../shared-references/datahub-cli-reference.md).

## Long queries — use a file

Long inline `--query` strings can trigger OS limits (e.g. “File name too long” on macOS). Write the operation to a `.graphql` file and pass the **path**:

```bash
datahub graphql --query /tmp/my-query.graphql --format json
```

The CLI treats paths and inline strings differently; prefer files for large selections.

## One operation per `.graphql` file (or disambiguate)

The CLI is easiest to use when each `--query` file contains **a single** GraphQL operation (one `query` / `mutation` / `subscription`). If a document defines **multiple** named operations, you may need **`--operation <Name>`** to pick which one to run (`datahub graphql --help`).

**Workflow for several operations:**

1. **Split** into separate files (`search.graphql`, `mutate.graphql`, …) and run `datahub graphql --query` once per file, or  
2. Use a **runner** (shell script, task, or CI) that loops over one-operation files and invokes the CLI for each — the same pattern as a `test-datahub-graphql-patterns.sh`-style harness.

Avoid stuffing many unrelated operations into one large `.graphql` unless you rely on `--operation` and keep names explicit.

## Cross-reference: writes and batch patterns

Tag, glossary, domain, and other mutations are documented with examples in [`../../shared-references/datahub-cli-reference.md`](../../shared-references/datahub-cli-reference.md) (Write Operations, GraphQL Discovery).

# UI-managed ingestion (GraphQL recipe read/update)

When metadata ingestion is configured in the **DataHub UI** (managed ingestion sources), recipes live in DataHub metadata—not only in local YAML files. You can **read**, **update**, and **trigger runs** through the **GraphQL API** using the same `datahub graphql` CLI as `/datahub-graphql`.

**Product overview:** [Metadata Ingestion (UI)](https://docs.datahub.com/docs/ui-ingestion)  
**Queries index:** [GraphQL queries](https://docs.datahub.com/docs/graphql/queries) (search for `Ingestion`, `Execution`)  
**Mutations index:** [GraphQL mutations](https://docs.datahub.com/docs/graphql/mutations) (search for `Ingestion`)

## When to use this path

- The user manages ingestion in **Settings → Manage Metadata Ingestion** (or equivalent) and does **not** use `datahub ingest -c` from a repo.
- The user provides an **ingestion source URN** or name that exists in the UI.

If the deployment **does not** expose ingestion GraphQL (no `listIngestionSources` / `updateIngestionSource` in `datahub graphql --list-operations`), fall back to **CLI + local recipe** only and say that UI ingestion APIs are unavailable on this server.

## Permissions

UI ingestion typically requires platform privileges such as **Manage Metadata Ingestion** (view/edit/execute per policy). If mutations fail with authorization errors, stop and ask an admin to confirm policies—do not retry blindly.

## Ground operations in the live schema

Do **not** assume field names beyond what appears in product docs for your server version. Before composing queries or mutations:

```bash
datahub -C skill=datahub-lineage-debug graphql --list-operations --format json
datahub -C skill=datahub-lineage-debug graphql --list-mutations --format json
datahub -C skill=datahub-lineage-debug graphql --describe listIngestionSources --recurse --format json
datahub -C skill=datahub-lineage-debug graphql --describe updateIngestionSource --recurse --format json
datahub -C skill=datahub-lineage-debug graphql --describe createIngestionExecutionRequest --recurse --format json
datahub -C skill=datahub-lineage-debug graphql --describe listExecutionRequests --recurse --format json
datahub -C skill=datahub-lineage-debug graphql --describe executionRequest --recurse --format json
datahub -C skill=datahub-lineage-debug graphql --describe ingestionSource --recurse --format json
```

Adjust operation names if your server uses different spelling—**Tier 1 introspection wins** (see `/datahub-graphql`).

## Run history and logs (queries)

Use these **read** operations to debug failed or stale runs (see [GraphQL queries](https://docs.datahub.com/docs/graphql/queries)):

| Goal | Query | Notes |
| --- | --- | --- |
| List runs across the deployment | `listExecutionRequests` | Input type `ListExecutionRequestsInput`—use `--describe` for filters (for example by ingestion source, time window, or state if supported). |
| One run by URN | `executionRequest` | Argument `urn`—returns `ExecutionRequest` with `input`, `result`, and related fields per schema. |
| History for a single ingestion source | `ingestionSource` | `IngestionSource` exposes paginated **execution requests** (often `executionRequests` / `IngestionSourceExecutionRequests` with `start`, `count`, `total`)—confirm subfields with `--describe ingestionSource --recurse`. |

### Where logs live on `ExecutionRequest`

`ExecutionRequestResult` (under the execution request’s **result**) typically includes:

- **Status / timing** (for example success vs failure, start time, duration—exact field names from `--describe`).
- **Reports:** a structured payload often modeled as **`StructuredReport`**: `type` (for example `INGESTION_REPORT`, `TEST_CONNECTION_REPORT`), **`serializedValue`** (string, usually JSON), and `contentType`. Parse **`serializedValue`** for ingestion stdout/stderr-style details, counters, and errors the executor returned.

Always select only the subfields you need; large `serializedValue` blobs can be heavy.

## Typical operations (verify with `--describe`)

| Goal | Direction | Names to look for (common in OSS docs) |
| --- | --- | --- |
| List sources | Query | `listIngestionSources` → `ListIngestionSourcesResult` / `ingestionSources` |
| List execution runs | Query | `listExecutionRequests` (return shape from `--describe`) |
| Get one run + reports / logs | Query | `executionRequest(urn:)` → `result` → reports / `StructuredReport.serializedValue` |
| Fetch source + run history | Query | `ingestionSource(urn:)` → nested `executionRequests` (paginated) |
| Read recipe/config | Query subfields | `IngestionSource` config; recipe is often a **string** (JSON-encoded recipe payload—confirm type in schema) |
| Apply recipe change | Mutation | `updateIngestionSource` (`urn` + `UpdateIngestionSourceInput`) |
| Run ingestion after change | Mutation | `createIngestionExecutionRequest` |
| Cancel run (if stuck) | Mutation | `cancelIngestionExecutionRequest` (if present) |

Official examples for **creating** sources with a JSON `recipe` string appear in the UI ingestion doc ([deploying recipes via GraphQL](https://docs.datahub.com/docs/ui-ingestion#deploying-recipes-graphql)). **Updating** follows the same idea: serialize the recipe, escape per GraphQL string rules, and pass through `updateIngestionSource` input fields described by `--describe`.

## Recipe handling

1. **Fetch** current config from the query result; **parse** the recipe string as JSON if the schema indicates JSON-in-string.
2. **Edit** the minimal JSON/YAML-equivalent structure (source/sink blocks) per connector docs (`connector-docs-howto.md`).
3. **Serialize** back to the form the mutation expects (often a single JSON string with proper escaping).
4. Use **`--variables` / a JSON file** for mutation variables so quotes and newlines in the recipe do not break the shell (same pattern as `/datahub-enrich` for complex URNs).

## Secrets

Recipes often reference **secret placeholders** (for example `${MYSQL_PASSWORD}`) managed in DataHub. Do **not** paste real secrets into chat or variables files the user might commit. Preserve placeholder names unless the user explicitly rotates secrets through the **Manage Secrets** flow documented in [UI ingestion](https://docs.datahub.com/docs/ui-ingestion).

## After a successful run

Repeat lineage checks (`datahub lineage` / MCP) as in the main skill—same verification as CLI ingestion.

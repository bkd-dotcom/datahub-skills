# Reading DataHub connector documentation (for lineage debug)

Official guide: [Connector Documentation — Structure & Authoring Guide](https://github.com/datahub-project/datahub/blob/master/metadata-ingestion/docs/sources/AGENTS.md) (`metadata-ingestion/docs/sources/AGENTS.md`).

## What to read

Only edit or rely on **hand-authored** files for narrative and examples:

| File | Role |
| --- | --- |
| `README.md` | Platform overview and concept mapping |
| `{connector}_pre.md` | Module sections before injected blocks (often Overview, Prerequisites) |
| `{connector}_post.md` | After injected blocks (often Capabilities, Limitations, Troubleshooting) |
| `{connector}_recipe.yml` | Minimal example recipe |

**Do not** treat `docs/generated/...` as something to hand-edit; it is build output. For **current** field names and valid config, use the **injected** sections on the published docs site or the **Config Details** / JSON schema that the doc pipeline assembles—per AGENTS.md, that material is generated and reflects the real schema.

## Canonical example

When structure or tone is unclear, use **Snowflake** as the reference layout: `snowflake/README.md`, `snowflake_pre.md`, `snowflake_recipe.yml`, `snowflake_post.md` under the same `docs/sources/` tree.

## Versioned links

When citing GitHub paths for a **specific** deployment:

1. Resolve the DataHub **release** (for example from `datahub check server-config`, image tag, or Helm chart version).
2. Link blobs with that **tag**, not unqualified `master` / `main`, unless you are speaking abstractly about the latest OSS tree.

**Link pattern:**

`https://github.com/datahub-project/datahub/blob/<TAG>/metadata-ingestion/docs/sources/<platform>/`

Replace `<platform>` with the connector folder name (for example `snowflake`, `dbt`, `airflow`, `looker`).

## Deprecated recipe settings

- Prefer **generated config schema** and **hand-authored** troubleshooting over copying keys from old internal recipes.
- If an old recipe disagrees with current docs/schema, treat **current docs + schema** as source of truth and update the recipe accordingly.

# URN mismatch playbook (cross-platform lineage)

Dataset URNs look like:

`urn:li:dataset:(urn:li:dataPlatform:<platform>,<qualified_name>,<env>)`

Other entity types (dashboard, chart, dataJob, dataFlow) have different URN shapes. When **upstream** and **downstream** metadata refer to the "same" object but lineage does not connect, work through this checklist.

## 1. Environment (`env`)

- **Symptom:** Identical platform and table name, different last segment of dataset URN.
- **Check:** `PROD` vs `PROD` casing, or `PROD` vs `DEV`, or missing `env` alignment between sources.
- **Fix direction:** Align `env` (or equivalent) across recipes for every source that should share an edge. Confirm per-connector docs for the correct key name.

## 2. Platform instance

- **Symptom:** One side uses `urn:li:dataPlatformInstance:(urn:li:dataPlatform:X, Y)` in metadata or qualified names differ because **instance** is folded into identity; edges emitted without instance do not match.
- **Check:** Whether `platform_instance` / `instance` (exact key per connector schema) is set on one recipe but not another, or set to different values.
- **Fix direction:** Make instance configuration **consistent** across all ingestion recipes that should link to the same logical datasets, per connector documentation.

## 3. Qualified name conventions

- **Symptom:** Same real table, different `<qualified_name>` string (catalog, database, schema, or table segment missing or reordered).
- **Common patterns:**
  - Warehouse: `db.schema.table` vs `schema.table`
  - Catalog + database prefixes (for example Iceberg, Glue, multi-catalog warehouses)
  - Case sensitivity: `My_Table` vs `MY_TABLE`
  - Extra quoting or escaped identifiers in lineage payloads
- **Fix direction:** Adjust **namespace** / **database** / **schema** / **table pattern** / **strip** options only as supported by the **specific** connector schema—never assume a key exists; verify in docs.

## 4. dbt vs warehouse (siblings)

- **Symptom:** User expects an edge on the **Snowflake** dataset URN; lineage is only on the **dbt** dataset URN (or reverse).
- **Check:** `siblings` aspect / search projection in `/datahub-lineage` or `/datahub-search`.
- **Fix direction:** May be **expected**—lineage is on dbt nodes while BI points at warehouse tables. Use sibling awareness when interpreting gaps; recipe changes may involve dbt **target** alignment, `env`, or instance—not always "wrong" if siblings link correctly.

## 5. Orchestration vs warehouse

- **Symptom:** Airflow/Dagster/Prefect lineage references dataset FQNs that do not match warehouse extractor naming.
- **Check:** DataJob / DataFlow upstreamLineage vs warehouse dataset URNs; SQL parser or `inlets`/`outlets` naming.
- **Fix direction:** Per orchestration connector docs: operator patterns, cluster / namespace, dataset alias templates, or lineage extraction flags.

## 6. BI tools (Looker, Tableau, etc.)

- **Symptom:** Dashboard points at an explore or field that resolves to a different qualified name than the warehouse dataset.
- **Check:** Explore / view / custom SQL mapping in the BI connector docs.
- **Fix direction:** Connection defaults, `connection_override`, or include/exclude patterns as documented for that source.

## 7. No lineage aspect at all

- **Symptom:** Search finds the dataset; `upstreamLineage` is empty or lineage returns zero edges.
- **Check:** Whether lineage is optional for that source and requires **capability flags**, **permissions** (for example query logs), or a **separate** lineage recipe.
- **Fix direction:** Enable documented lineage features; ensure prerequisites (roles, warehouses, audit tables) from `*_pre.md` / Troubleshooting.

## 8. Stale metadata

- **Symptom:** Old edges remain after pipeline renames.
- **Check:** Stateful ingestion / soft-delete behavior in connector docs.
- **Fix direction:** May require stateful cleanup or re-ingest with corrected patterns—not always a URN typo.

Use this playbook together with **connector-specific** troubleshooting in `*_post.md` and versioned schema for the deployment.

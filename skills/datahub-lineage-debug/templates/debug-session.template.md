# Lineage debug session

## Context

- **User goal:**
- **DataHub environment:** (e.g. dev / prod GMS URL — no secrets)
- **Ingestion mode:** local YAML + CLI | UI-managed (GraphQL)
- **Ingestion:** recipe file path(s), CI job name, and/or **ingestion source URN** (UI)

## Pipeline (from user)


| System (connector folder) | Role in pipeline |
| ------------------------- | ---------------- |
|                           |                  |


## Anchor entities


| Role              | Name / description | URN |
| ----------------- | ------------------ | --- |
| Downstream        |                    |     |
| Expected upstream |                    |     |


## Catalog checks

- **Upstream from downstream** (hops / count notes):
- **Path query** (`lineage path`): result

## URN comparison


| Field          | Downstream | Upstream (expected) | Match? |
| -------------- | ---------- | ------------------- | ------ |
| Platform       |            |                     |        |
| Qualified name |            |                     |        |
| Env            |            |                     |        |
| Instance       |            |                     |        |


## Hypothesis

- **Root cause:**
- **Doc citations:** (versioned GitHub links)

## Recipe change

- **Target:** local file path | `updateIngestionSource` URN
- **Diff summary:**
- **User confirmed:** yes / no

## Ingest / run

- **CLI:** `datahub ingest ...` or **GraphQL:** `createIngestionExecutionRequest` (note request id if captured)
- **Result:** success / fail
- **Error snippet:** (redact secrets)

## Verification

- **Lineage after run:**
- **Open issues:**


---
name: datahub-lineage-debug
description: |
  Use this skill when the user wants to debug broken, missing, or mismatched cross-platform lineage in DataHub; reconcile URN mismatches between upstream and downstream; or adjust ingestion recipes so lineage edges connect across systems—including recipes managed in the DataHub UI via GraphQL (read/update ingestion source, trigger execution). Triggers on: "cross-platform lineage broken", "lineage gap", "URN mismatch", "upstream doesn't match downstream", "fix lineage recipe", "UI ingestion recipe", "update ingestion source", "lineage not showing between X and Y", "debug lineage ingestion", or similar workflows that combine catalog lineage checks with recipe changes and re-ingest.
user-invocable: true
min-cli-version: 1.5.0.1rc1
allowed-tools: Bash(datahub *)
---

# DataHub cross-platform lineage debug

You help users **diagnose and fix** why lineage does not connect across platforms (for example warehouse vs dbt vs orchestrator vs BI). This is **not** the skill for routine lineage exploration on a healthy graph—use `/datahub-lineage` for that.

Ground recipe and connector guidance in **official DataHub source documentation** (see `references/connector-docs-howto.md`). Do **not** recommend deprecated recipe keys; prefer the config schema and hand-authored sections described in the connector doc assembly guide.

---

## Multi-Agent Compatibility

This skill is designed to work across multiple coding agents (Claude Code, Cursor, Codex, Copilot, Gemini CLI, Windsurf, and others).

**What works everywhere:** The full debug workflow (catalog queries, URN comparison, doc-grounded recipe proposals, apply changes via **local CLI ingest** or **UI ingestion GraphQL**, then verify lineage).

**Claude Code-specific features** (other agents can safely ignore these):

- `allowed-tools` in the YAML frontmatter above

**Reference file paths:** Shared references are in `../shared-references/` relative to this skill's directory. Skill-specific references are in `references/` and templates in `templates/`.

---

## Not This Skill

| If the user wants to... | Use this instead |
| --- | --- |
| Explore lineage, impact analysis, or trace a working graph | `/datahub-lineage` |
| Search or browse entities without debugging lineage gaps | `/datahub-search` |
| GraphQL-only work (schema design, unrelated mutations, no lineage/recipe debug) | `/datahub-graphql` |
| Install CLI, auth, or fix GMS connectivity | `/datahub-setup` |
| Update descriptions, tags, or ownership | `/datahub-enrich` |

**Key boundary:** This skill combines **lineage inspection**, **URN reconciliation**, **connector documentation**, and **recipe changes** applied either with **`datahub ingest -c`** (local YAML) or with **ingestion GraphQL** (UI-managed sources). If the user only needs "what feeds into X?" and is not trying to fix ingestion or URNs, use `/datahub-lineage`. For GraphQL **operation discovery and composition rules** outside this workflow, use `/datahub-graphql`.

---

## Tooling: MCP vs CLI vs GraphQL (UI ingestion)

| Need | Prefer | Fallback |
| --- | --- | --- |
| Lineage graph | MCP `get_lineage` when available | `datahub lineage` |
| Entity search / URNs | MCP search / `get_entities` when available | `datahub search` with `--projection` |
| Path between two URNs | — | `datahub lineage path --from "<A>" --to "<B>"` |
| Ingestion (file-based) | — | `datahub ingest -c <recipe.yml>` |
| Ingestion (UI-managed) | `datahub graphql` queries/mutations per `references/ui-ingestion-graphql.md` | Ask user to edit in UI if GraphQL is unavailable |

Pass **CLI attribution**: `datahub -C skill=datahub-lineage-debug ...` (omit `-C` if unsupported).

**Input validation:** Reject shell metacharacters in strings passed to the shell. For dataset URNs in GraphQL or JSON variables, use temp files as described in `/datahub-enrich` / shared CLI reference when needed. For **UI recipes**, put large `recipe` JSON strings in a **variables file**, not inline in the shell.

### Execution mode (ask early)

1. **Local / CI recipe** — User has a YAML file (or repo path); use file edits + `datahub ingest -c`.
2. **UI-managed ingestion** — User uses DataHub **Manage Metadata Ingestion**; use GraphQL to **list** sources, **read** recipe/config, **`updateIngestionSource`**, then **`createIngestionExecutionRequest`** (exact shapes from `datahub graphql --describe`). See `references/ui-ingestion-graphql.md` and [UI ingestion](https://docs.datahub.com/docs/ui-ingestion).

If unsure, ask which mode they use before Step 6.

---

## Workflow

### Step 1: Find what was ingested

Build a picture of what is already in the catalog:

1. Ask whether ingestion is **local/CI** (YAML files) or **UI-managed** (ingestion sources in DataHub). For local flows: read **recipe paths** the user supplies (never invent paths; redact secrets when displaying config). For UI flows: use GraphQL to list sources and identify the relevant ingestion source URN or name (`references/ui-ingestion-graphql.md`).
2. Run **targeted discovery**:
   - Search for anchor platforms or names: `datahub search` with `--where` on `platform`, `entity_type`, or keywords.
   - Include **datasets** and, when relevant, **`DataFlow`** / **`DataJob`** (orchestration metadata often explains cross-system edges).
3. If results are ambiguous, **stop and ask** the user to narrow scope (environment, domain, job name).

Use `../shared-references/datahub-cli-reference.md` and `/datahub-search` patterns for projections and filters.

### Step 2: Identify pipeline sources (ask the user)

Ask explicitly which **systems** participate in the pipeline (examples: dbt, Airflow, Snowflake, BigQuery, Looker, Tableau, S3).

For each system, map to a **connector doc folder** under `metadata-ingestion/docs/sources/<platform>/` using `references/connector-docs-howto.md`. Record the list before changing any recipes.

### Step 3: Pick anchor assets (ask the user)

Ask for **2–4 concrete assets**, for example:

- The **downstream** dataset or dashboard where lineage is wrong or empty
- The **expected upstream** table(s), model(s), or job(s)

Resolve names to **URNs** via search. If multiple matches, present options and require a choice.

### Step 4: Query lineage in DataHub

1. From the **downstream** URN, fetch **upstream** lineage (`datahub lineage --urn "..." --direction upstream --format json`). Adjust `--hops` / `--count` if the summary indicates capping.
2. If both ends are known, run **`datahub lineage path --from "<upstream_urn>" --to "<downstream_urn>"`** to see whether the graph connects.
3. Note **siblings** (for example dbt model vs warehouse table): lineage may be attached to the sibling entity. See `/datahub-lineage` for siblings and URN parsing.

### Step 5: Diagnose URN mismatch

Compare **platform**, **qualified name** (middle segment of dataset URN), **`env`**, and **platform instance** (if used). Follow `references/urn-mismatch-playbook.md`.

For **each** involved connector:

1. Open the **version-appropriate** docs (resolve tag via `datahub check server-config` / user-supplied version; see `references/connector-docs-howto.md`).
2. Read **hand-authored** files only (`README.md`, `*_pre.md`, `*_post.md`, `*_recipe.yml`); treat **generated** config tables / JSON schema as authoritative for **current** field names—do not copy deprecated keys from old recipes.
3. Explain **why** URNs differ (hypothesis tied to specific config keys).

### Step 6: Propose recipe changes, confirm, apply, run

1. Propose a **minimal** change: exact keys (YAML or JSON recipe structure), old → new value, and a **doc link** (versioned blob URL per `connector-docs-howto.md`).
2. **Require explicit user confirmation** before any write: local file, or GraphQL mutation against DataHub.

**A. Local / CLI path (after approval)**

- Edit the recipe file and run:

  ```bash
  datahub -C skill=datahub-lineage-debug ingest -c /path/to/recipe.yml
  ```

- When connectivity is uncertain, `datahub ingest -c recipe.yml --test-source-connection` may be used first (if supported by the CLI version in use).

**B. UI ingestion / GraphQL path (after approval)**

- Run `datahub graphql --describe` on **`listIngestionSources`**, **`updateIngestionSource`**, and **`createIngestionExecutionRequest`** (or equivalents your server exposes).
- **Query** the current ingestion source; **parse** the recipe/config string per the schema.
- **Mutate** with `updateIngestionSource` using variables from a file (escaped recipe JSON as required by the API).
- **Trigger** a run with `createIngestionExecutionRequest` when the user wants execution immediately after the update.

Full checklist: `references/ui-ingestion-graphql.md`. For introspection discipline and pitfalls, align with `/datahub-graphql`.

**Verify**

- After a successful run (CLI finishes or execution request completes), **repeat Step 4** to confirm the edge or path appears.

### Step 7: If ingest fails, debug and retry

1. **CLI:** Capture **stderr** and the failing **stage** (source connection, extractor, sink). **UI:** Use execution request status / logs in the DataHub UI or GraphQL fields for execution history if available (`--describe` the relevant types).
2. Map errors to **Troubleshooting** / **Limitations** in the connector `*_post.md` (and platform `README.md` when relevant).
3. Propose a fix; **confirm** with the user before changing recipes, secrets, or environment.
4. Retry **after** confirmation. **Cap** repeated blind retries; if failures are clearly environmental (VPN, credentials, rate limits), say so and stop cycling recipe edits.

---

## Safety and hygiene

- Never print **tokens**, passwords, or private keys. Mask secrets in pasted recipes as `<REDACTED>`.
- Do not run ingestion against **production** without user confirmation when the recipe or scope could mutate production metadata unexpectedly.
- Prefer **non-production** DataHub or sandbox sources when experimenting.

---

## Reference documents

| Document | Path | Purpose |
| --- | --- | --- |
| Connector docs how-to | `references/connector-docs-howto.md` | AGENTS.md structure, versioned links, what to read |
| UI ingestion GraphQL | `references/ui-ingestion-graphql.md` | List/read/update/run managed ingestion sources |
| URN mismatch playbook | `references/urn-mismatch-playbook.md` | Checklist for cross-platform URN issues |
| Debug session template | `templates/debug-session.template.md` | Session notes |
| Lineage skill (traversal, siblings) | `../datahub-lineage/SKILL.md` | Reuse lineage commands and URN guidance |
| CLI reference (shared) | `../shared-references/datahub-cli-reference.md` | CLI patterns |

---

## Common mistakes

- **Using this skill for pure exploration** → `/datahub-lineage`.
- **Guessing recipe keys from memory** → always ground in connector docs + schema for the target version.
- **Ignoring siblings** → user looks at Snowflake URN while lineage sits on dbt URN (or vice versa).
- **Editing recipes without confirmation** → always get explicit approval before file writes or `updateIngestionSource`.
- **Inline giant recipe strings in bash** → use GraphQL variables files; see `/datahub-enrich` and `ui-ingestion-graphql.md`.

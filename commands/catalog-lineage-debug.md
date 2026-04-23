---
name: catalog-lineage-debug
description: Debug broken cross-platform lineage, URN mismatches, and ingestion recipes
argument-hint: "[lineage gap or systems involved]"
---

# DataHub lineage debug

Use the Skill tool to invoke the full `datahub-lineage-debug` skill:

```
Skill tool:
  skill: "datahub-skills:datahub-lineage-debug"
```

**User's request:** $ARGUMENTS

This skill diagnoses missing or broken lineage across platforms:

1. Discover what is ingested in the catalog (local recipes and/or UI-managed ingestion sources via GraphQL)
2. Confirm pipeline systems with the user
3. Anchor on specific assets (URNs)
4. Query upstream lineage and paths
5. Compare URNs and ground fixes in official connector docs (per AGENTS.md)
6. Propose recipe changes, confirm, then apply via **local `datahub ingest`** or **GraphQL** (`updateIngestionSource` + run request when using UI ingestion)
7. On failure, troubleshoot and retry after confirmation

If no arguments are provided, ask what lineage gap the user sees and which systems are involved.

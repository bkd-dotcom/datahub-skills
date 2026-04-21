---
name: catalog-graphql
description: Build or validate DataHub GraphQL queries and mutations; introspect the schema
argument-hint: "[query, mutation, or feasibility question]"
---

# DataHub GraphQL

Use the Skill tool to invoke the full `datahub-graphql` skill:

```
Skill tool:
  skill: "datahub-skills:datahub-graphql"
```

**User's request:** $ARGUMENTS

This skill helps with programmatic DataHub GraphQL:

1. Discover operations with `datahub graphql --list-operations` / `--describe … --recurse`
2. Compose queries and mutations grounded in the live schema (no invented field names)
3. Validate shapes with `--describe`; there is no `graphql --dry-run` — executing `--query` calls the server (use safe reads or non-prod for mutations)
4. Answer whether GraphQL supports a given pattern on your server

If no arguments provided, ask what GraphQL operation or API question the user needs.

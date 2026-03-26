---
title: Ingestion
slug: ingestion
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-23
---

# Ingestion

> Ingestion is the automated process by which CoreStory analyzes a source code repository and produces the Intelligence Model — transforming raw code into structured, queryable Code Intelligence.

## Definition

Ingestion is the pipeline that takes a connected code repository as input, analyzes its structure, behavior, and business logic, and produces the Intelligence Model as output. It is automated and does not require developers to write descriptions, annotations, or documentation. The result is a fully populated Intelligence Model ready for querying.

Ingestion is a point-in-time operation. The Intelligence Model reflects the state of the repository at the time ingestion ran. Re-ingestion is required to incorporate new changes.

## Where It Applies

- Initial setup: connecting a repository to CoreStory for the first time
- Periodic refresh: re-ingesting after significant code changes
- CI/CD integration: triggering re-ingestion automatically on merge to main

## Where It Does NOT Apply

- Real-time code monitoring (ingestion is not continuous streaming)
- Binary files, compiled artifacts, or non-source assets
- Repositories with no readable source code (e.g., pure config repos with no logic)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| Ingestion requires manual tagging or annotation | Ingestion is fully automated — no human input is required |
| Ingestion produces documentation | Ingestion produces machine-readable Code Intelligence and Specs, not human-facing docs |
| Ingestion is instantaneous | Ingestion time scales with repository size; large repos may take minutes |
| You only ingest once | Re-ingestion is expected as the codebase evolves |

## Example

A team connects their Node.js API repository to CoreStory. CoreStory ingests the repository: it parses the module structure, traces function call graphs, extracts business rules from conditional logic, and maps domain concepts across files. Within minutes, the Intelligence Model is populated and ready. A developer can immediately query: "What are all the places that modify user account status?" and receive a structured answer — without having written any documentation.

## Related Pages

- [→ Intelligence Model](../intelligence-model)
- [→ Code Intelligence](../code-intelligence)
- [→ Codebase Analysis](../codebase-analysis)

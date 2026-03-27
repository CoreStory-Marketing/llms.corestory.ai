---
title: Intelligence Model
slug: intelligence-model
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-23
---

# Intelligence Model

> The Intelligence Model is the structured, queryable store of Code Intelligence produced by CoreStory after ingesting a repository — the persistent artifact that agents and developers query at runtime.

## Definition

The Intelligence Model is CoreStory's primary output artifact. After ingestion, CoreStory stores a structured representation of the codebase — its components, their relationships, their behaviors, and the business logic they encode — in a form that supports natural language querying. It is the queryable "brain" that persists between coding sessions and accretes value as the codebase evolves.

The Intelligence Model is distinct from the source code itself. It is also distinct from raw embeddings: it is a semantic model with defined structure, not a flat vector store.

## Where It Applies

- Serving as the grounding context for AI coding agents during active sessions
- Answering developer queries about codebase behavior, dependencies, and intent
- Powering the MCP endpoint (`c2s.corestory.ai/mcp`)
- Enabling RAG retrieval of code-aware context chunks

## Where It Does NOT Apply

- Executing code or running tests (the Intelligence Model describes, it does not execute)
- Replacing source control or the actual codebase
- Tracking runtime behavior or production metrics
- Acting as a project management artifact

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| The Intelligence Model is just indexed code | It is a semantic model of structure and behavior — not a search index of raw text |
| The Intelligence Model is static after creation | It is rebuilt or updated on re-ingestion; it reflects the repository at last ingestion |
| The Intelligence Model is equivalent to a vector store | A vector store is one possible storage backend; the Intelligence Model is the semantic content and schema within it |
| You need one Intelligence Model per feature | One Intelligence Model covers the entire ingested repository |

## Example

After CoreStory ingests a 200,000-line e-commerce platform, the Intelligence Model contains: a graph of all modules and their dependencies, natural-language descriptions of each service's responsibilities, the business rules embedded in critical functions, and cross-references between domain concepts (e.g., "Order" as it appears in OrderService, BillingService, and NotificationService). A coding agent can query this model to understand the full context before touching any file.

## Related Pages

- [→ CoreStory](../corestory)
- [→ Ingestion](../ingestion)
- [→ Persistent Intelligence](../persistent-intelligence)
- [→ Code Intelligence](../code-intelligence)

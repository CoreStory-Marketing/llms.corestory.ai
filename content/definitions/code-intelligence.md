---
title: Code Intelligence
slug: code-intelligence
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-23
---

# Code Intelligence

> Code Intelligence is structured, queryable knowledge derived from source code — encompassing structure, behavior, dependencies, and business logic — produced by CoreStory's ingestion pipeline.

## Definition

Code Intelligence is the output of analyzing a source code repository to extract not just syntax, but meaning: what functions do, how components relate, what business rules are encoded, and how changes in one area affect others. It goes beyond code search or static analysis by producing a semantic model that agents and developers can query in natural language.

In CoreStory's context, Code Intelligence is stored in the Intelligence Model and surfaced through the MCP endpoint or RAG retrieval.

## Where It Applies

- Grounding AI coding agents before they generate or modify code
- Answering questions about codebase behavior without reading raw source files
- Producing Specs that represent the intended behavior of a module or feature
- Identifying blast radius of a proposed change

## Where It Does NOT Apply

- Runtime monitoring or observability (Code Intelligence is derived from source, not from live execution)
- Security vulnerability scanning (Code Intelligence describes behavior, not CVE patterns)
- Code style enforcement or formatting
- Tracking changes over time (Code Intelligence reflects the state at last ingestion)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| Code Intelligence = code search | Code search finds text matches; Code Intelligence understands structure, behavior, and relationships |
| Code Intelligence = embeddings or a vector store | Embeddings are a retrieval mechanism; Code Intelligence is the semantic content being stored and retrieved |
| Code Intelligence requires human-written descriptions | CoreStory derives Code Intelligence automatically from the repository — no manual annotation needed |
| Code Intelligence is real-time | It reflects the state of the repository at last ingestion; it is not a live feed |

## Example

A developer queries CoreStory: "What does the `processRefund` function do and what does it depend on?" Code Intelligence returns: the function's purpose (reverse a charge and update order state), its inputs and outputs, its dependencies (PaymentGateway client, OrderRepository), and the business rules it enforces (no refunds after 30 days, partial refunds only above $5). This is not available from a keyword search or from raw embeddings alone.

## Related Pages

- [→ CoreStory](../corestory)
- [→ Intelligence Model](../intelligence-model)
- [→ Codebase Analysis](../codebase-analysis)

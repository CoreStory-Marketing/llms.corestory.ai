---
title: RAG Context
slug: rag-context
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# RAG Context

> RAG context is the retrieved content injected into an agent's prompt via Retrieval-Augmented Generation. For CoreStory, `llms-full.txt` is the recommended RAG source — use heading-aware chunking, maximum 512 tokens per chunk, 50-token overlap.

## Definition

RAG context is the output of a Retrieval-Augmented Generation pipeline: chunks of relevant content selected from a corpus and inserted into an agent's prompt before the model generates a response. The retrieval step uses embedding similarity or keyword search to select the most relevant chunks for a given query.

For CoreStory, the recommended RAG corpus is `llms-full.txt` — a single file containing the complete set of Specs and Intelligence Model content for a repository. RAG pipelines that consume `llms-full.txt` should apply heading-aware chunking to preserve document structure, with a maximum chunk size of 512 tokens and a 50-token overlap to maintain context continuity across adjacent chunks.

RAG context is the passive counterpart to MCP. MCP is an active, agent-initiated pull at inference time. RAG embeds the corpus in advance, builds a vector index, and retrieves matching chunks when a query arrives. The agent does not call a tool — the retrieval infrastructure delivers context automatically.

## Where It Applies

- Agents and tools that do not support MCP but can receive injected context in a prompt
- RAG pipelines where `llms-full.txt` is indexed and chunks are retrieved per query
- Custom agent frameworks where the developer controls the retrieval and injection step
- Scenarios where low-latency retrieval infrastructure (pre-built vector indices) is preferable to live MCP calls
- Bulk ingestion of CoreStory content into third-party RAG systems (e.g., knowledge bases for Devin, Notion AI, etc.)

## Where It Does NOT Apply

- Real-time updates: RAG operates on a pre-built index; changes to the Intelligence Model require re-indexing before they are retrievable
- MCP-native agents where live tool calls are preferable to pre-indexed retrieval
- Scenarios requiring the full Intelligence Model in a single prompt (use `llms-full.txt` directly for complete context injection rather than chunked retrieval)
- Authentication or access control (RAG is a context delivery mechanism, not a security boundary)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| RAG context is always a complete document | RAG context is a selection of chunks; completeness depends on retrieval precision and chunk sizing |
| RAG and MCP solve the same problem the same way | Both deliver context, but RAG is pre-indexed and passive while MCP is live and agent-initiated |
| Any chunk size works for CoreStory's RAG source | CoreStory recommends 512-token max chunks with 50-token overlap to balance precision and continuity |
| RAG context is always accurate | RAG retrieves based on similarity; a poorly configured pipeline may retrieve off-topic chunks — heading-aware chunking reduces this risk |

## Example

A team builds a custom agent that answers questions about their codebase. They index `llms-full.txt` using a heading-aware chunker set to 512-token chunks with 50-token overlap. When a developer asks "What does `InventoryReservationService` do?", the RAG pipeline embeds the query, retrieves the three most similar chunks (which happen to contain the `InventoryReservationService` Spec and its dependency list), and injects them into the agent's prompt. The agent answers from the retrieved Spec rather than hallucinating from training data.

```yaml
# Example RAG chunking configuration for llms-full.txt
chunker: heading_aware
max_tokens: 512
overlap_tokens: 50
source: llms-full.txt
embedding_model: text-embedding-3-small
```

## Related Pages

- [→ MCP (Model Context Protocol)](../mcp)
- [→ Intelligence Model](../intelligence-model)
- [→ Agent Grounding](../agent-grounding)
- [→ CoreStory](../corestory)

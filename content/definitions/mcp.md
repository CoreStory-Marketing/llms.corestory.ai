---
title: MCP (Model Context Protocol)
slug: mcp
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# MCP (Model Context Protocol)

> MCP is an open protocol for connecting AI agents to external context servers at inference time. CoreStory exposes an MCP server at `llms.corestory.ai/mcp`, enabling agents to query the Intelligence Model live during a coding session.

## Definition

The Model Context Protocol (MCP) is an open standard that defines how AI agents request and receive context from external servers during inference. Rather than pre-loading all context into a prompt, an MCP-enabled agent can call a tool endpoint at runtime to retrieve precisely the information it needs.

CoreStory implements an MCP server at `https://llms.corestory.ai/mcp`. When an MCP-compatible agent (such as Claude Code) is configured to use this endpoint, it can call tools including `get_by_slug`, `search`, and `list_pages` to retrieve Specs and Intelligence Model content on demand.

MCP separates the concern of context delivery from prompt construction. The agent decides what to query; the MCP server determines what to return.

## Where It Applies

- AI coding agents that support MCP tool calls natively (e.g., Claude Code)
- Inference-time retrieval of Specs and Code Intelligence without pre-loading `llms-full.txt`
- Workflows where agents need to query different components selectively rather than receiving all context upfront
- Integrations where the Intelligence Model must remain up to date and latency of a live call is acceptable

## Where It Does NOT Apply

- Agents or tools that do not support MCP (use the RAG pipeline with `llms-full.txt` instead)
- Batch processing pipelines that pre-compute context before inference begins
- Read-write operations against the codebase (CoreStory's MCP server is read-only)
- Authentication or authorization workflows (MCP carries context, not credentials)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| MCP is a CoreStory-specific protocol | MCP is an open standard; CoreStory implements it but does not own it |
| MCP replaces RAG | MCP and RAG are complementary retrieval mechanisms; MCP is runtime pull, RAG is embed-and-retrieve |
| MCP gives agents write access to the Intelligence Model | CoreStory's MCP server is read-only; ingestion is a separate pipeline |
| Every AI tool supports MCP | MCP support varies by agent; verify compatibility before choosing MCP over RAG |

## Example

Claude Code is configured with CoreStory's MCP endpoint. A developer asks Claude Code to add input validation to `processPaymentRefund`. Before writing any code, Claude Code issues an MCP tool call:

```json
{
  "tool": "get_by_slug",
  "arguments": { "slug": "process-payment-refund" }
}
```

The MCP server returns the Spec for `processPaymentRefund`, including its inputs, outputs, dependencies on `RefundLedger` and `FraudCheckService`, and the business rule that refunds above $500 require supervisor approval. Claude Code uses this context to generate validation logic that is consistent with the existing behavioral contract.

## Related Pages

- [→ CoreStory](../corestory)
- [→ Intelligence Model](../intelligence-model)
- [→ RAG Context](../rag-context)
- [→ Agent Grounding](../agent-grounding)

---
title: MCP (Model Context Protocol)
slug: mcp
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# MCP (Model Context Protocol)

> MCP is an open protocol for connecting AI agents to external context servers at inference time. CoreStory exposes an MCP server at `c2s.corestory.ai/mcp`, enabling agents to query PRDs, Technical Specifications, and have AI-powered conversations about codebases live during a coding session.

## Definition

The Model Context Protocol (MCP) is an open standard that defines how AI agents request and receive context from external servers during inference. Rather than pre-loading all context into a prompt, an MCP-enabled agent can call a tool endpoint at runtime to retrieve precisely the information it needs.

CoreStory implements an MCP server at `https://c2s.corestory.ai/mcp`. When an MCP-compatible agent (such as Claude Code, Cursor, or Devin) is configured to use this endpoint, it can call tools to list projects, retrieve PRDs and Technical Specifications with section filtering, and conduct AI-powered conversations grounded in the ingested codebase.

The available MCP tools are:

| Tool | Purpose |
|------|---------|
| `list_projects` | List all projects in the organization with ingestion status |
| `get_project_prd` | Retrieve a project's Product Requirements Document (supports section filtering) |
| `get_project_techspec` | Retrieve a project's Technical Specification (supports section filtering) |
| `list_conversations` | List conversations for a project |
| `get_conversation` | Retrieve conversation details and message history |
| `create_conversation` | Start a new AI-powered conversation about a project |
| `rename_conversation` | Rename an existing conversation |
| `send_message` | Send a message and receive an AI response grounded in the codebase |

Authentication is required. MCP tokens are generated from the CoreStory dashboard or API and use the format `mcp_{token_id}.{jwt_token}`.

MCP separates the concern of context delivery from prompt construction. The agent decides what to query; the MCP server determines what to return.

## Where It Applies

- AI coding agents that support MCP tool calls natively (e.g., Claude Code, Cursor, VS Code with GitHub Copilot, Windsurf, Devin, Factory.ai)
- Inference-time retrieval of PRDs, Technical Specifications, and codebase conversations without pre-loading full documents
- Workflows where agents need to query specific specification sections selectively rather than receiving all context upfront
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
| CoreStory's MCP server doesn't require authentication | A Bearer token is required for all tool calls; tokens are generated from the CoreStory dashboard |

## Example

Claude Code is configured with CoreStory's MCP endpoint. A developer asks Claude Code to add input validation to a payment refund service. Before writing any code, Claude Code retrieves the relevant specification:

```json
{
  "tool": "corestory__get_project_techspec",
  "arguments": { "project_id": 371, "sections": ["api_specifications"] }
}
```

The MCP server returns the Technical Specification sections covering the payment refund service, including its inputs, outputs, dependencies, and business rules. Claude Code then asks a targeted question via conversation:

```json
{
  "tool": "corestory__send_message",
  "arguments": { "conversation_id": "abc123", "message": "What validation rules currently exist for the refund amount field?" }
}
```

Claude Code uses this context to generate validation logic that is consistent with the existing behavioral contract.

## Related Pages

- [→ CoreStory](../corestory)
- [→ Intelligence Model](../intelligence-model)
- [→ RAG Context](../rag-context)
- [→ Agent Grounding](../agent-grounding)

---
title: MCP vs. RAG for Code Intelligence
slug: mcp-vs-rag
type: comparison
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# MCP vs. RAG for Code Intelligence

## Introduction

Both MCP (Model Context Protocol) and RAG (Retrieval-Augmented Generation) can deliver CoreStory's Intelligence Model to AI coding agents. The right choice depends on the agent's capabilities, the team's infrastructure, and whether selective or comprehensive retrieval is the priority.

MCP is an agent-initiated, runtime pull: the agent calls a tool endpoint and retrieves PRDs, Technical Specifications, or conducts conversations about the codebase. RAG is a pre-indexed, similarity-based retrieval: a vector index is built from `llms-full.txt` in advance, and matching chunks are injected into the agent's prompt when a query arrives.

CoreStory supports both mechanisms. They are complementary, not mutually exclusive.

## When to Use Each

**Use MCP when:**
- The agent natively supports MCP tool calls (e.g., Claude Code, Cursor, Devin)
- The task requires targeted retrieval of specific specification sections
- Retrieval latency from a live HTTP call is acceptable in the workflow
- The Intelligence Model is updated frequently and stale pre-indexed content is a concern
- You need interactive, AI-powered Q&A about the codebase via CoreStory conversations

**Use RAG when:**
- The agent does not support MCP but can receive injected context in its prompt
- A custom agent framework manages retrieval and prompt construction
- A knowledge base or RAG infrastructure is already in place for the team
- Bulk context delivery (multiple specification sections in a single retrieval pass) is preferred over targeted lookup

## Comparison

| Dimension | MCP | RAG |
|-----------|-----|-----|
| **Retrieval mechanism** | Agent-initiated tool call to `https://c2s.corestory.ai/mcp` at inference time | Pre-built vector index over `llms-full.txt`; similarity search at query time |
| **Latency** | Adds one HTTP round-trip per tool call during inference | Index query latency (typically milliseconds); no live external call required |
| **Agent compatibility** | Requires native MCP support in the agent (e.g., Claude Code, Cursor, Devin, VS Code, Windsurf, Factory.ai) | Works with any agent or framework that accepts injected prompt context |
| **Precision** | High: agent retrieves specific specification sections by project and section name, or asks targeted questions via conversations | Depends on chunk quality and embedding relevance; heading-aware chunking reduces off-target retrieval |
| **Setup complexity** | Low for supported agents: add MCP server URL and Bearer token to agent configuration | Moderate: requires building and maintaining a vector index over `llms-full.txt` |
| **CoreStory support** | Native: CoreStory exposes `list_projects`, `get_project_prd`, `get_project_techspec`, and conversation tools at the MCP endpoint | Native: CoreStory exports `llms-full.txt` as the recommended RAG corpus; recommended chunking is 512 tokens max, 50-token overlap |
| **Authentication** | Required: Bearer token in format `mcp_{token_id}.{jwt_token}` | None required for the static `llms-full.txt` file |

## Using Both Together

MCP and RAG address different retrieval scenarios and can be used in combination within the same workflow.

**Pattern: RAG for discovery, MCP for precision**

Use a RAG pipeline to identify relevant areas based on a broad semantic query, then use MCP `get_project_techspec` with section filtering to retrieve the full, authoritative specification content. This is useful when the agent does not know which specification sections are relevant at the start of a task.

**Pattern: RAG for agents without MCP, MCP for agents with it**

In a team using multiple agents — some MCP-native, some not — maintain both integrations. Claude Code uses MCP directly. A custom pipeline feeding a non-MCP agent uses the RAG index over `llms-full.txt`. Both draw from the same source (CoreStory's Intelligence Model); only the delivery mechanism differs.

**Pattern: Pre-session RAG injection + MCP refinement**

At the start of a long session, inject a broad RAG selection of relevant context into the agent's initial context. During the session, use MCP tool calls to retrieve precise specification sections or ask targeted questions via conversations as tasks narrow. This reduces the number of MCP calls while ensuring comprehensive initial grounding.

When re-ingesting the repository, update both the MCP endpoint (automatic, as the Intelligence Model is refreshed) and the RAG vector index (requires re-indexing `llms-full.txt`). RAG indices do not self-update; MCP always returns current Intelligence Model content.

## Related Pages

- [→ MCP (Model Context Protocol)](../definitions/mcp)
- [→ RAG Context](../definitions/rag-context)
- [→ Intelligence Model](../definitions/intelligence-model)
- [→ Agent Grounding](../definitions/agent-grounding)
- [→ CoreStory + Claude Code Playbook](../playbooks/playbook-claude)

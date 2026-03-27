---
title: CoreStory + Claude Code
slug: playbook-claude
type: integration-guide
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory + Claude Code

## Overview

Claude Code is an MCP-native AI coding agent. It supports runtime tool calls to external MCP servers, making it the highest-fidelity integration path for CoreStory. When connected to CoreStory's MCP server, Claude Code can retrieve PRDs, Technical Specifications, and have AI-powered conversations about your codebase on demand — querying only what is relevant to the current task rather than pre-loading an entire codebase description.

This Playbook covers the steps to configure the integration, verify it, and use it in a Spec-Driven Development workflow.

## Prerequisites

- Claude Code installed and functional in your terminal environment
- An active CoreStory account with at least one ingested project
- A CoreStory MCP token (generated from the dashboard or API)

## Step 1: Generate an MCP Token

1. Navigate to **IDE Integrations** in your CoreStory dashboard settings.
2. Enter an optional token name (e.g., "MacBook Pro — Claude Code").
3. Click **Generate** and copy the token immediately — it will not be shown again.
4. The token format is `mcp_{token_id}.{jwt_token}`.

Alternatively, generate a token via the API at `https://c2s.corestory.ai/docs`.

## Step 2: Connect CoreStory's MCP Server

Add the CoreStory MCP server to Claude Code using the CLI:

```bash
claude mcp add --transport http corestory https://c2s.corestory.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_TOKEN_HERE"
```

Restart Claude Code after adding the server.

## Step 3: Verify the Connection

After restarting, verify that Claude Code can reach the CoreStory MCP server by invoking the `list_projects` tool:

```
Use the corestory list_projects tool to show my available projects.
```

Claude Code will issue the following tool call:

```json
{
  "tool": "corestory__list_projects",
  "arguments": {}
}
```

Expected response: a list of projects in your organization with their ingestion status, including `has_prd` and `has_techspec` flags. If the call fails, verify your token and endpoint configuration.

## Step 4: Retrieve a PRD or Tech Spec Before Editing Code

Before modifying any component, retrieve the relevant specification for the project. CoreStory provides two document types: **PRDs** (product requirements with user stories) and **Technical Specifications** (architecture, API contracts, data models).

First, discover available sections:

```
Use corestory get_project_techspec for project ID 371 with sections_only set to true.
```

Then retrieve the sections relevant to your task:

```
Use corestory get_project_techspec for project ID 371 with sections set to ["api_specifications", "data_models"].
```

The returned content includes detailed architectural context, business rules, and behavioral contracts that ground Claude Code's reasoning before it generates or modifies code.

## Step 5: Have a Conversation About the Codebase

For exploratory questions or complex investigations, use CoreStory's conversation feature. This provides AI-powered responses grounded in the ingested codebase.

```
Use corestory create_conversation for project ID 371 with the title "Payment refund investigation".
```

Then ask questions:

```
Use corestory send_message to the conversation we just created, asking "What business rules govern the payment refund process, and what dependencies does it have?"
```

CoreStory's AI responds with answers grounded in the actual codebase, not generic training data.

## Step 6: Spec-Driven Development Workflow

A complete Spec-Driven Development (SDD) cycle with Claude Code follows this sequence:

**Task**: Add a maximum refund limit of $1,000 to the payment refund service.

**1. Retrieve the relevant specification:**

```
Use corestory get_project_techspec for project 371, sections ["api_specifications"].
```

**2. Ask CoreStory about the specific component:**

```
Use corestory send_message asking "What are the business rules and dependencies for the payment refund service?"
```

**3. Issue the coding task, grounded in the spec:**

```
Based on the CoreStory specification above, add a hard limit: refunds may not exceed $1,000 regardless of order total. This check should occur before the supervisor approval check.
```

**4. Verify against the spec:**

```
Review the changes you made. Confirm that:
- The $1,000 cap is enforced before the supervisor approval check.
- The existing business rules from the spec are unchanged.
- Audit logging is still called regardless of outcome.
```

This sequence — retrieve spec, ground the task, generate code, verify against spec — is the core SDD loop.

## Best Practices

- Retrieve the relevant PRD or Tech Spec sections for every component Claude Code will touch before issuing any coding instruction.
- Use `sections_only: true` first to discover available sections, then request only the sections you need — this keeps context focused and avoids overloading the prompt.
- Use CoreStory conversations for exploratory questions when you need to understand how components interact before deciding what to change.
- After re-ingestion updates the Intelligence Model, re-fetch specs rather than relying on specs retrieved in earlier sessions.
- Use `list_projects` at the start of a new coding session to orient Claude Code to the available projects and their ingestion status.
- Use a separate MCP token for each device so you can revoke access individually if needed.

## Common Pitfalls

- **Skipping spec retrieval for "small" changes**: Even single-line changes can violate business rules that are invisible in the source code but visible in the specification. Always retrieve context first.
- **Using a stale spec**: If the codebase has been modified since the last CoreStory ingestion, the spec may not reflect current behavior. Re-ingest before critical sessions.
- **Forgetting to authenticate**: Unlike some MCP servers, CoreStory requires a Bearer token. If tool calls return authentication errors, verify your token is valid and not expired.

## Related Pages

- [→ MCP (Model Context Protocol)](../../definitions/mcp)
- [→ Agent Grounding](../../definitions/agent-grounding)
- [→ Spec](../../definitions/spec)
- [→ Spec-Driven Development (SDD)](../../definitions/sdd)
- [→ Playbook (definition)](../../definitions/playbook)

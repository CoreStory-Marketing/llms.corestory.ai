---
title: CoreStory + Cursor
slug: playbook-cursor
type: integration-guide
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory + Cursor

## Overview

Cursor supports MCP server connections and project-level rules files, providing two complementary integration paths for CoreStory. MCP allows Cursor's agent to query PRDs, Technical Specifications, and have AI-powered conversations about the codebase at inference time via tool calls. The `.cursorrules` file allows persistent, project-scoped instructions that can instruct Cursor to use CoreStory as its grounding source for all coding tasks in the repository.

This Playbook covers both integration methods and the workflow for using them together.

## Prerequisites

- Cursor version 0.40 or later (MCP support is required for tool call integration)
- An active CoreStory account with at least one ingested project
- A CoreStory MCP token (generated from the dashboard or API)
- Write access to the repository root for `.cursorrules` configuration

## Step 1: Generate an MCP Token

1. Navigate to **IDE Integrations** in your CoreStory dashboard settings.
2. Enter an optional token name (e.g., "Cursor — Work Laptop").
3. Click **Generate** and copy the token immediately — it will not be shown again.
4. The token format is `mcp_{token_id}.{jwt_token}`.

## Step 2: Connect CoreStory's MCP Server

Open Cursor's settings and navigate to **Tools & MCP**. Add the CoreStory MCP server configuration.

Cursor stores MCP server configuration in `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` at the repository root (project-scoped). Use the project-scoped file to limit the integration to repositories with a CoreStory Intelligence Model.

```json
{
  "mcpServers": {
    "corestory": {
      "url": "https://c2s.corestory.ai/mcp",
      "type": "http",
      "headers": {
        "Authorization": "Bearer mcp_YOUR_TOKEN_HERE"
      }
    }
  }
}
```

Restart Cursor after saving. The CoreStory server should appear in the MCP panel with a green status indicator.

## Step 3: Verify the Connection

In Cursor's Chat panel, invoke the `list_projects` tool to confirm the connection is active:

```
@corestory list all available projects
```

Cursor will call:

```json
{
  "tool": "corestory__list_projects",
  "arguments": {}
}
```

A successful response returns a list of projects with their ingestion status and document availability flags (`has_prd`, `has_techspec`). If the call fails, check the MCP configuration URL, authentication headers, and Cursor's MCP log output in the developer console.

## Step 4: Retrieve a Spec Before Editing Code

Before modifying a component, retrieve the relevant specification using `@corestory` in Cursor Chat.

First, discover available sections:

```
@corestory get the tech spec for project 371, sections only
```

Then retrieve the relevant sections:

```
@corestory get the tech spec for project 371, sections: api_specifications, data_models
```

Include the returned specification in the task prompt to ground Cursor's reasoning:

```
Here is the tech spec for the inventory module:
[paste the returned spec]

Add retry logic for transient database failures. Do not modify the reservation timeout or the business rule that reservations expire after 15 minutes.
```

## Step 5: Configure .cursorrules for Persistent Grounding

Add a `.cursorrules` file to the repository root to instruct Cursor to use CoreStory for all coding tasks in this project. This ensures grounding behavior persists across sessions without requiring developers to remember to query the MCP server manually.

```
# .cursorrules

## CoreStory Integration

This repository uses CoreStory for Code Intelligence. Before modifying any component:

1. Query the CoreStory MCP server using @corestory list_projects to identify the relevant project.
2. Use @corestory get_project_techspec or get_project_prd with section filtering to retrieve context for the component you will modify.
3. For exploratory questions, use @corestory create_conversation and send_message to investigate the codebase.
4. Do not modify behavior described in the specification without explicit instruction to do so.

CoreStory MCP endpoint: https://c2s.corestory.ai/mcp
Available tools: list_projects, get_project_prd, get_project_techspec, list_conversations, get_conversation, create_conversation, rename_conversation, send_message

## Spec-Driven Development

All code changes in this repository follow Spec-Driven Development (SDD). This means:
- Agent outputs must be consistent with the retrieved specification.
- Business rules described in the spec are constraints, not suggestions.
- After making changes, verify that all spec-listed dependencies are still correctly used.
```

## Step 6: Spec-Driven Development Workflow

A complete SDD cycle with Cursor:

**Task**: Reduce the inventory reservation timeout from 15 to 10 minutes.

**1. Retrieve the specification:**

```
@corestory get the tech spec for project 371, sections: api_specifications
```

**2. Ask CoreStory about the component:**

```
@corestory send a message to conversation asking "What are the business rules and dependencies for the inventory reservation service?"
```

**3. Issue the grounded task:**

```
Based on the CoreStory specification, change the reservation timeout from 15 minutes to 10 minutes. Update the constant definition and any inline comments that reference the 15-minute rule. Do not change the cleanup job polling frequency or the reservation limit per session.
```

**4. Verify:**

```
Review the changes. Confirm the timeout is 10 minutes, existing business rules are unchanged, and all dependencies are unmodified.
```

## Best Practices

- Add `.cursorrules` to every repository using CoreStory so that all developers on the team benefit from consistent grounding behavior.
- Use project-scoped MCP configuration (`.cursor/mcp.json`) rather than global configuration to avoid activating CoreStory in repositories that have not been ingested.
- Use `sections_only: true` to discover available sections before fetching content — this avoids pulling oversized documents into the context window.
- Retrieve specs at the start of each Cursor Chat session — do not assume a prior session's retrieved specs are still current.
- When working on a feature that spans multiple components, retrieve all relevant spec sections before issuing any code generation task.

## Common Pitfalls

- **MCP server not appearing in Cursor**: Verify the `mcp.json` file is valid JSON and that Cursor has been restarted since the file was saved. Check the MCP panel in Cursor settings for error messages.
- **`.cursorrules` instructions being ignored**: Cursor applies `.cursorrules` at session start — if the file is added mid-session, restart the Chat session for the rules to take effect.
- **Authentication errors**: CoreStory requires a valid Bearer token. If tool calls fail, verify your token is not expired and is correctly formatted in the `headers` block.

## Related Pages

- [→ MCP (Model Context Protocol)](../../definitions/mcp)
- [→ Agent Grounding](../../definitions/agent-grounding)
- [→ Spec](../../definitions/spec)
- [→ Spec-Driven Development (SDD)](../../definitions/sdd)
- [→ Playbook (definition)](../../definitions/playbook)

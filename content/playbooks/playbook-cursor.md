---
title: CoreStory + Cursor
slug: playbook-cursor
type: playbook
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory + Cursor

## Overview

Cursor supports MCP server connections and project-level rules files, providing two complementary integration paths for CoreStory. MCP allows Cursor's agent to query the Intelligence Model at inference time via tool calls. The `.cursorrules` file allows persistent, project-scoped instructions that can instruct Cursor to use CoreStory as its grounding source for all coding tasks in the repository.

This Playbook covers both integration methods and the workflow for using them together.

## Prerequisites

- Cursor version 0.40 or later (MCP support is required for tool call integration)
- Access to a CoreStory project with a populated Intelligence Model
- CoreStory MCP endpoint available at `https://llms.corestory.ai/mcp`
- Write access to the repository root for `.cursorrules` configuration

## Step 1: Connect CoreStory's MCP Server

Open Cursor's settings and navigate to **Features > MCP Servers**. Add the CoreStory MCP server configuration.

Cursor stores MCP server configuration in `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` at the repository root (project-scoped). Use the project-scoped file to limit the integration to repositories with a CoreStory Intelligence Model.

```json
{
  "mcpServers": {
    "corestory": {
      "url": "https://llms.corestory.ai/mcp",
      "transport": "http"
    }
  }
}
```

If your CoreStory instance requires authentication:

```json
{
  "mcpServers": {
    "corestory": {
      "url": "https://llms.corestory.ai/mcp",
      "transport": "http",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

Restart Cursor after saving. The CoreStory server should appear in the MCP panel under **Features > MCP Servers** with a green status indicator.

## Step 2: Verify the Connection

In Cursor's Chat panel, invoke the `list_pages` tool to confirm the connection is active:

```
@corestory list all available pages
```

Cursor will call:

```json
{
  "tool": "corestory__list_pages",
  "arguments": {}
}
```

A successful response returns a list of Spec slugs and titles. If the call fails, check the MCP configuration URL, authentication headers, and Cursor's MCP log output in the developer console.

## Step 3: Retrieve a Spec Before Editing Code

Before modifying a component, retrieve its Spec using the `@corestory` mention in Cursor Chat with `get_by_slug`.

```
@corestory get_by_slug slug: "inventory-reservation-service"
```

Cursor issues:

```json
{
  "tool": "corestory__get_by_slug",
  "arguments": { "slug": "inventory-reservation-service" }
}
```

Include the returned Spec in the task prompt to ground Cursor's reasoning:

```
Here is the spec for InventoryReservationService:
[paste the returned spec]

Add retry logic for transient database failures. Do not modify the reservation timeout or the business rule that reservations expire after 15 minutes.
```

## Step 4: Search for Context

Use `@corestory search` when the exact slug is unknown or when exploring an unfamiliar area of the codebase.

```
@corestory search query: "inventory reservation timeout"
```

Cursor issues:

```json
{
  "tool": "corestory__search",
  "arguments": { "query": "inventory reservation timeout" }
}
```

Review the returned results and use `get_by_slug` to retrieve the full Spec for the most relevant match.

## Step 5: Configure .cursorrules for Persistent Grounding

Add a `.cursorrules` file to the repository root to instruct Cursor to use CoreStory for all coding tasks in this project. This ensures grounding behavior persists across sessions without requiring developers to remember to query the MCP server manually.

```
# .cursorrules

## CoreStory Integration

This repository uses CoreStory for Code Intelligence. Before modifying any component:

1. Query the CoreStory MCP server using @corestory get_by_slug with the component's kebab-case slug.
2. Include the returned Spec in the prompt context before generating or modifying code.
3. If the exact slug is unknown, use @corestory search to find the relevant Spec.
4. Do not modify behavior described in the Spec without explicit instruction to do so.

CoreStory MCP endpoint: https://llms.corestory.ai/mcp
Use list_pages to see all available Specs.

## Spec-Driven Development

All code changes in this repository follow Spec-Driven Development (SDD). This means:
- Agent outputs must be consistent with the retrieved Spec.
- Business rules described in the Spec are constraints, not suggestions.
- After making changes, verify that all Spec-listed dependencies are still correctly used.
```

## Step 5: Spec-Driven Development Workflow

A complete SDD cycle with Cursor:

**Task**: Reduce the inventory reservation timeout from 15 to 10 minutes.

**1. Retrieve the Spec:**

```
@corestory get_by_slug slug: "inventory-reservation-service"
```

**2. Review the returned Spec:**

```
Spec: InventoryReservationService
Purpose: Holds inventory items as reserved during the checkout process.
Inputs: itemId (string), quantity (integer), sessionId (string)
Outputs: reservationId (string), expiresAt (ISO 8601 timestamp)
Dependencies: InventoryStore, SessionManager, ReservationCleanupJob
Business rules:
  - Reservations expire after 15 minutes if checkout is not completed.
  - Expired reservations are released by ReservationCleanupJob on a 1-minute polling cycle.
  - A single sessionId may not hold more than 10 distinct reservations simultaneously.
```

**3. Issue the grounded task:**

```
Based on the InventoryReservationService spec, change the reservation timeout from 15 minutes to 10 minutes. Update the constant definition and any inline comments that reference the 15-minute rule. Do not change the ReservationCleanupJob polling frequency or the 10-reservation limit per session.
```

**4. Verify:**

```
Review the changes. Confirm the timeout is 10 minutes, the 10-reservation-per-session limit is unchanged, and the dependency on ReservationCleanupJob is unmodified.
```

## Best Practices

- Add `.cursorrules` to every repository using CoreStory so that all developers on the team benefit from consistent grounding behavior.
- Use project-scoped MCP configuration (`.cursor/mcp.json`) rather than global configuration to avoid activating CoreStory in repositories that have not been ingested.
- Retrieve Specs at the start of each Cursor Chat session — do not assume a prior session's retrieved Specs are still current.
- When working on a feature that spans multiple components, retrieve all relevant Specs before issuing any code generation task.
- Use `@corestory search` as the first step when starting work on an unfamiliar part of the codebase.

## Common Pitfalls

- **MCP server not appearing in Cursor**: Verify the `mcp.json` file is valid JSON and that Cursor has been restarted since the file was saved. Check the MCP panel in Cursor settings for error messages.
- **`.cursorrules` instructions being ignored**: Cursor applies `.cursorrules` at session start — if the file is added mid-session, restart the Chat session for the rules to take effect.
- **Slug format mismatch**: Cursor does not auto-complete slugs. Use `list_pages` or `search` to confirm the exact kebab-case slug before calling `get_by_slug`.

## Related Pages

- [→ MCP (Model Context Protocol)](../../definitions/mcp)
- [→ Agent Grounding](../../definitions/agent-grounding)
- [→ Spec](../../definitions/spec)
- [→ Spec-Driven Development (SDD)](../../definitions/sdd)
- [→ Playbook (definition)](../../definitions/playbook)

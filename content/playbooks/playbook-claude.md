---
title: CoreStory + Claude Code
slug: playbook-claude
type: playbook
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory + Claude Code

## Overview

Claude Code is an MCP-native AI coding agent. It supports runtime tool calls to external MCP servers, making it the highest-fidelity integration path for CoreStory. When connected to CoreStory's MCP endpoint, Claude Code can retrieve Specs and Intelligence Model content on demand — querying only what is relevant to the current task rather than pre-loading an entire codebase description.

This Playbook covers the steps to configure the integration, verify it, and use it in a Spec-Driven Development workflow.

## Prerequisites

- Claude Code installed and functional in your terminal environment
- Access to a CoreStory project with a populated Intelligence Model
- CoreStory MCP endpoint available at `https://llms.corestory.ai/mcp`
- (Optional) A project-specific API key if your CoreStory instance requires authentication

## Step 1: Connect CoreStory's MCP Server

Add the CoreStory MCP server to Claude Code's MCP configuration file. The configuration file is typically located at `~/.claude/mcp_servers.json` or specified via the `--mcp-config` flag.

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

If your CoreStory instance requires an API key, add it as a header:

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

Restart Claude Code after saving the configuration.

## Step 2: Verify the Connection

After restarting, verify that Claude Code can reach the CoreStory MCP server by invoking the `list_pages` tool. In Claude Code, you can trigger this directly:

```
Use the corestory list_pages tool to show available pages.
```

Claude Code will issue the following tool call:

```json
{
  "tool": "corestory__list_pages",
  "arguments": {}
}
```

Expected response: a list of slugs and titles representing the available Specs and content pages in the Intelligence Model. If the list is empty or the call fails, verify the URL and authentication configuration.

## Step 3: Retrieve a Spec Before Editing Code

Before modifying any component, retrieve its Spec using `get_by_slug`. The slug corresponds to the component name in kebab-case.

Example prompt to Claude Code:

```
Use corestory get_by_slug with slug "payment-refund-service" and include the result in your context before making any changes.
```

Claude Code issues:

```json
{
  "tool": "corestory__get_by_slug",
  "arguments": { "slug": "payment-refund-service" }
}
```

The returned Spec includes the component's purpose, inputs, outputs, dependencies, and business rules. Instruct Claude Code to reason from this Spec before generating or modifying code:

```
Given the following spec for PaymentRefundService, add input validation for the `amount` field. Do not change any behavior described in the spec.

[paste or reference the returned spec here]
```

## Step 4: Search for Context

When you do not know the exact slug, use the `search` tool to find relevant Specs by keyword or concept.

```
Use corestory search with query "rate limiting email" to find relevant specs.
```

Claude Code issues:

```json
{
  "tool": "corestory__search",
  "arguments": { "query": "rate limiting email" }
}
```

The response returns matching Specs ranked by relevance. Select the most relevant result and follow Step 3 to retrieve its full content.

## Step 5: Spec-Driven Development Workflow

A complete Spec-Driven Development (SDD) cycle with Claude Code follows this sequence:

**Task**: Add a maximum refund limit of $1,000 to `PaymentRefundService`.

**1. Query the spec:**

```
Use corestory get_by_slug with slug "payment-refund-service".
```

**2. Review the returned spec:**

```
Spec: PaymentRefundService
Purpose: Processes customer refund requests against completed orders.
Inputs: orderId (string), amount (float, USD), requestedBy (string, user ID)
Outputs: refundConfirmation (object: refundId, status, processedAt)
Dependencies: OrderLedger, FraudCheckService, RefundAuditLog
Business rules:
  - Refunds may not exceed the original order total.
  - Refunds above $500 require supervisor approval (FraudCheckService.requiresReview).
  - All refunds are logged to RefundAuditLog regardless of outcome.
```

**3. Issue the coding task, grounded in the spec:**

```
Based on the PaymentRefundService spec above, add a hard limit: refunds may not exceed $1,000 regardless of order total. This check should occur before the FraudCheckService.requiresReview check. Update the spec comment in the code to reflect the new rule.
```

**4. Verify against the spec:**

```
Review the changes you made. Confirm that:
- The $1,000 cap is enforced before the supervisor approval check.
- The existing business rules from the spec are unchanged.
- RefundAuditLog is still called regardless of outcome.
```

This sequence — query spec, ground the task, generate code, verify against spec — is the core SDD loop.

## Best Practices

- Retrieve the Spec for every component Claude Code will touch before issuing any coding instruction.
- Include the full text of the retrieved Spec in the prompt context, not just the slug — Claude Code reasons better with the content than with a reference to it.
- Use `search` when working in unfamiliar areas of the codebase to surface related Specs before narrowing to a specific component.
- After re-ingestion updates the Intelligence Model, re-fetch Specs rather than relying on Specs retrieved in earlier sessions.
- Use the `list_pages` tool at the start of a new coding session to orient Claude Code to the available Intelligence Model content.

## Common Pitfalls

- **Skipping Spec retrieval for "small" changes**: Even single-line changes can violate business rules that are invisible in the source code but visible in the Spec. Always retrieve the Spec first.
- **Using a stale Spec**: If the codebase has been modified since the last CoreStory ingestion, the Spec may not reflect current behavior. Re-ingest before critical sessions.
- **Querying with a wrong slug format**: Slugs are kebab-case (e.g., `payment-refund-service`, not `PaymentRefundService`). Use `search` or `list_pages` to confirm the correct slug if `get_by_slug` returns empty.

## Related Pages

- [→ MCP (Model Context Protocol)](../../definitions/mcp)
- [→ Agent Grounding](../../definitions/agent-grounding)
- [→ Spec](../../definitions/spec)
- [→ Spec-Driven Development (SDD)](../../definitions/sdd)
- [→ Playbook (definition)](../../definitions/playbook)

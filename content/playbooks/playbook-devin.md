---
title: CoreStory + Devin
slug: playbook-devin
type: playbook
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory + Devin

## Overview

Devin is an autonomous AI software engineer that executes multi-step coding tasks within a sandboxed environment. CoreStory integrates with Devin through three mechanisms: knowledge base upload (for persistent, session-available context), system prompt injection (for task-scoped grounding using `llms-full.txt`), and MCP tool use within Devin's agent loop (for precise, on-demand Spec retrieval during task execution).

For most teams, the recommended primary integration is knowledge base upload supplemented by system prompt injection. MCP tool use within Devin's agent loop is available for advanced configurations where selective Spec retrieval is preferred over pre-loaded context.

## Prerequisites

- A Devin account with access to the Knowledge or Playbooks configuration panel
- A CoreStory project with a populated Intelligence Model
- The `llms-full.txt` file exported from CoreStory for the target repository
- CoreStory MCP endpoint available at `https://llms.corestory.ai/mcp` (for MCP integration path)

## Step 1: Upload llms-full.txt to Devin's Knowledge Base

Devin's knowledge base allows you to upload documents that Devin can reference during any session. Upload `llms-full.txt` to give Devin persistent access to the full Intelligence Model.

1. Navigate to **Settings > Knowledge** in the Devin dashboard.
2. Click **Add Document**.
3. Upload the `llms-full.txt` file exported from CoreStory.
4. Set the document title to: `CoreStory Intelligence Model — [Repository Name]`
5. Set the description to: `Machine-generated Specs and Code Intelligence for [repository name]. Use this document to ground all coding tasks before generating or modifying code.`

Devin will automatically chunk and index the document. After indexing, Devin can retrieve relevant sections of the Intelligence Model when they are semantically related to the current task.

For repositories with large Intelligence Models, re-export and re-upload `llms-full.txt` after each CoreStory re-ingestion to keep Devin's knowledge base current.

## Step 2: Verify the Knowledge Base Integration

Create a new Devin session and issue a verification task:

```
Using the CoreStory Intelligence Model document in your knowledge base, describe what [ComponentName] does, including its inputs, outputs, and any business rules.
```

Devin should return a description that matches the Spec content in `llms-full.txt`. If Devin returns a generic or hallucinated response, check that the document was indexed successfully in the Knowledge panel.

## Step 3: Inject a Spec via System Prompt for Task-Scoped Grounding

For tasks where precise grounding is critical, inject the relevant Spec directly into the task's system prompt rather than relying solely on knowledge base retrieval. This ensures Devin has the exact Spec in its working context for the duration of the task.

When creating a Devin session or Playbook task, prepend the Spec content to the task description:

```
## Grounding Context

The following Spec was retrieved from CoreStory's Intelligence Model for the component you will modify:

---
Spec: OrderFulfillmentService
Purpose: Coordinates the end-to-end fulfillment workflow for confirmed orders, including warehouse picking, shipping label generation, and customer notification.
Inputs: orderId (string), fulfillmentPriority (enum: STANDARD | EXPRESS | OVERNIGHT)
Outputs: fulfillmentRecord (object: fulfillmentId, warehouseId, estimatedShipDate, trackingNumber)
Dependencies: WarehouseAllocationService, ShippingLabelService, CustomerNotificationService, FulfillmentAuditLog
Business rules:
  - EXPRESS and OVERNIGHT orders must be allocated to the nearest available warehouse.
  - STANDARD orders use the lowest-cost warehouse allocation.
  - CustomerNotificationService must be called after every successful fulfillment, regardless of priority.
  - All fulfillment attempts are written to FulfillmentAuditLog, including failures.
---

## Task

Add support for a new fulfillmentPriority value: SAME_DAY. SAME_DAY orders follow the same warehouse allocation logic as EXPRESS. They require a 2-hour cutoff check: if the current time is after 14:00 local warehouse time, the order must be deferred to the next business day and the customer notified via CustomerNotificationService with status DEFERRED.
```

This pattern gives Devin both the behavioral contract of the existing component and the precise task description.

## Step 4: Configure MCP Tool Use in Devin's Agent Loop

Devin supports MCP tool connections for agents that can call external tools during task execution. To enable CoreStory MCP tool use within Devin's agent loop:

1. Navigate to **Settings > Integrations > MCP Servers** in the Devin dashboard.
2. Add the CoreStory MCP server:

```json
{
  "name": "corestory",
  "url": "https://llms.corestory.ai/mcp",
  "transport": "http"
}
```

3. If authentication is required, add the API key header in the integration settings.

Once configured, Devin's agent loop can call CoreStory MCP tools during task execution. Include an instruction in the task prompt to trigger this behavior:

```
Before modifying any component, use the corestory get_by_slug tool to retrieve the relevant Spec. Use the retrieved Spec as grounding context for all code changes.
```

Devin will issue tool calls such as:

```json
{
  "tool": "corestory__get_by_slug",
  "arguments": { "slug": "order-fulfillment-service" }
}
```

and incorporate the returned Spec into its reasoning before generating code.

## Step 5: Spec-Driven Development Workflow

A complete SDD cycle with Devin using knowledge base + system prompt injection:

**Task**: Add a SAME_DAY fulfillment priority to `OrderFulfillmentService`.

**1. Export the current Spec from CoreStory:**

Use CoreStory's dashboard or MCP endpoint to retrieve the `order-fulfillment-service` Spec and copy its content.

**2. Create a Devin session with the Spec injected:**

Paste the Spec as grounding context at the top of the task (as shown in Step 3).

**3. Specify the task precisely:**

```
Add SAME_DAY to the fulfillmentPriority enum. Implement the 14:00 cutoff check using the warehouse's local timezone. Deferred orders must call CustomerNotificationService with status=DEFERRED. Do not change behavior for STANDARD, EXPRESS, or OVERNIGHT priorities. All fulfillment attempts must still be written to FulfillmentAuditLog.
```

**4. Review Devin's output against the Spec:**

After Devin completes the task, verify that:
- Existing business rules for STANDARD, EXPRESS, and OVERNIGHT are unchanged.
- FulfillmentAuditLog is called for SAME_DAY attempts (both successful and deferred).
- CustomerNotificationService is called for all SAME_DAY outcomes.

## Best Practices

- Re-upload `llms-full.txt` to Devin's knowledge base after every CoreStory re-ingestion. Stale knowledge base content is a leading cause of Devin producing code inconsistent with the current codebase.
- For critical tasks, inject the Spec directly in the system prompt rather than relying on knowledge base retrieval. Retrieval is probabilistic; direct injection is deterministic.
- Use Devin Playbooks to encode the system prompt injection pattern as a reusable template for your team's most common coding workflows.
- When using MCP tool use, verify in Devin's task log that `get_by_slug` was called before code generation — not after.
- Scope each Devin session to a single component or feature boundary. Broad sessions with many components are harder to ground effectively.

## Common Pitfalls

- **Knowledge base document not indexed**: Devin may take several minutes to index an uploaded document. Do not start a session immediately after upload — wait for the indexing confirmation.
- **Relying solely on knowledge base retrieval for complex tasks**: Knowledge base retrieval selects chunks by similarity; for tasks involving multiple interacting components, supplement with direct Spec injection to ensure all relevant context is present.
- **Devin modifying behavior described in the Spec without instruction**: If Devin changes existing behavior, check whether the task description was ambiguous or conflicting with the Spec. Explicit constraints in the task prompt ("do not change X") prevent this.

## Related Pages

- [→ MCP (Model Context Protocol)](../../definitions/mcp)
- [→ RAG Context](../../definitions/rag-context)
- [→ Agent Grounding](../../definitions/agent-grounding)
- [→ Spec](../../definitions/spec)
- [→ Spec-Driven Development (SDD)](../../definitions/sdd)
- [→ Playbook (definition)](../../definitions/playbook)

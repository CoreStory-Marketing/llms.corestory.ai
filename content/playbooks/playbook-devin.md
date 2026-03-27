---
title: CoreStory + Devin
slug: playbook-devin
type: integration-guide
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory + Devin

## Overview

Devin is an autonomous AI software engineer that executes multi-step coding tasks within a sandboxed environment. CoreStory integrates with Devin through three mechanisms: knowledge base upload (for persistent, session-available context), system prompt injection (for task-scoped grounding), and MCP tool use within Devin's agent loop (for on-demand retrieval of PRDs, Technical Specifications, and AI-powered conversation during task execution).

For most teams, the recommended primary integration is knowledge base upload supplemented by system prompt injection. MCP tool use within Devin's agent loop is available for advanced configurations where selective spec retrieval is preferred over pre-loaded context.

## Prerequisites

- A Devin account with access to the Knowledge or Playbooks configuration panel
- An active CoreStory account with at least one ingested project
- A CoreStory MCP token (generated from the dashboard or API)
- The `llms-full.txt` file from llms.corestory.ai (for knowledge base upload path)

## Step 1: Upload llms-full.txt to Devin's Knowledge Base

Devin's knowledge base allows you to upload documents that Devin can reference during any session. Upload `llms-full.txt` to give Devin persistent access to CoreStory conceptual context.

1. Navigate to **Settings > Knowledge** in the Devin dashboard.
2. Click **Add Document**.
3. Upload the `llms-full.txt` file from llms.corestory.ai.
4. Set the document title to: `CoreStory Intelligence Model — [Repository Name]`
5. Set the description to: `CoreStory Code Intelligence reference. Use this document to understand CoreStory concepts and workflows when grounding coding tasks.`

Devin will automatically chunk and index the document. After indexing, Devin can retrieve relevant sections when they are semantically related to the current task.

## Step 2: Verify the Knowledge Base Integration

Create a new Devin session and issue a verification task:

```
Using the CoreStory document in your knowledge base, describe what Spec-Driven Development is and how CoreStory enables it.
```

Devin should return a description consistent with CoreStory's conceptual framework. If Devin returns a generic or hallucinated response, check that the document was indexed successfully in the Knowledge panel.

## Step 3: Inject Spec Content via System Prompt for Task-Scoped Grounding

For tasks where precise grounding is critical, retrieve the relevant specification from CoreStory (via the dashboard, MCP, or API) and inject it directly into the task's system prompt. This ensures Devin has the exact specification in its working context for the duration of the task.

When creating a Devin session or Playbook task, prepend the spec content to the task description:

```
## Grounding Context

The following specification was retrieved from CoreStory for the component you will modify:

---
Component: OrderFulfillmentService
Purpose: Coordinates the end-to-end fulfillment workflow for confirmed orders, including warehouse picking, shipping label generation, and customer notification.
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

Devin supports MCP tool connections for agents that can call external tools during task execution.

1. Navigate to **Settings > MCP Marketplace > "Add Your Own"** in the Devin dashboard.
2. Set Transport to **HTTP**.
3. Set URL to `https://c2s.corestory.ai/mcp`.
4. Create a secret named `$API_TOKEN` with your CoreStory MCP token value.
5. Set the Authorization header to `Bearer $API_TOKEN`.

Once configured, Devin's agent loop can call CoreStory MCP tools during task execution. The available tools are:

| Tool | Purpose |
|------|---------|
| `list_projects` | List all projects in the organization |
| `get_project_prd` | Retrieve a project's PRD (supports section filtering) |
| `get_project_techspec` | Retrieve a project's Technical Specification (supports section filtering) |
| `list_conversations` | List conversations for a project |
| `get_conversation` | Retrieve conversation details and history |
| `create_conversation` | Start a new conversation about a project |
| `rename_conversation` | Rename an existing conversation |
| `send_message` | Send a message and receive an AI response grounded in the codebase |

Include an instruction in the task prompt to trigger MCP usage:

```
Before modifying any component, use the corestory get_project_techspec tool to retrieve the relevant specification sections. Use the retrieved spec as grounding context for all code changes.
```

## Step 5: Spec-Driven Development Workflow

A complete SDD cycle with Devin using knowledge base + system prompt injection:

**Task**: Add a SAME_DAY fulfillment priority to `OrderFulfillmentService`.

**1. Retrieve the current spec from CoreStory:**

Use CoreStory's dashboard, MCP endpoint, or API to retrieve the relevant Technical Specification sections for the fulfillment service.

**2. Create a Devin session with the spec injected:**

Paste the spec as grounding context at the top of the task (as shown in Step 3).

**3. Specify the task precisely:**

```
Add SAME_DAY to the fulfillmentPriority enum. Implement the 14:00 cutoff check using the warehouse's local timezone. Deferred orders must call CustomerNotificationService with status=DEFERRED. Do not change behavior for STANDARD, EXPRESS, or OVERNIGHT priorities. All fulfillment attempts must still be written to FulfillmentAuditLog.
```

**4. Review Devin's output against the spec:**

After Devin completes the task, verify that:
- Existing business rules for STANDARD, EXPRESS, and OVERNIGHT are unchanged.
- FulfillmentAuditLog is called for SAME_DAY attempts (both successful and deferred).
- CustomerNotificationService is called for all SAME_DAY outcomes.

## Best Practices

- Re-upload `llms-full.txt` to Devin's knowledge base when the content is updated. Stale knowledge base content is a leading cause of Devin producing code inconsistent with the current codebase.
- For critical tasks, inject the spec directly in the system prompt rather than relying on knowledge base retrieval. Retrieval is probabilistic; direct injection is deterministic.
- Use Devin Playbooks to encode the system prompt injection pattern as a reusable template for your team's most common coding workflows.
- When using MCP tool use, verify in Devin's task log that spec retrieval was called before code generation — not after.
- Scope each Devin session to a single component or feature boundary. Broad sessions with many components are harder to ground effectively.
- Use a dedicated MCP token for Devin so you can revoke access independently of other tools.

## Common Pitfalls

- **Knowledge base document not indexed**: Devin may take several minutes to index an uploaded document. Do not start a session immediately after upload — wait for the indexing confirmation.
- **Relying solely on knowledge base retrieval for complex tasks**: Knowledge base retrieval selects chunks by similarity; for tasks involving multiple interacting components, supplement with direct spec injection to ensure all relevant context is present.
- **Devin modifying behavior described in the spec without instruction**: If Devin changes existing behavior, check whether the task description was ambiguous or conflicting with the spec. Explicit constraints in the task prompt ("do not change X") prevent this.

## Related Pages

- [→ MCP (Model Context Protocol)](../../definitions/mcp)
- [→ RAG Context](../../definitions/rag-context)
- [→ Agent Grounding](../../definitions/agent-grounding)
- [→ Spec](../../definitions/spec)
- [→ Spec-Driven Development (SDD)](../../definitions/sdd)
- [→ Playbook (definition)](../../definitions/playbook)

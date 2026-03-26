---
title: Agent Grounding
slug: agent-grounding
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# Agent Grounding

> Agent grounding is the process of providing an AI coding agent with accurate, relevant context before it generates or modifies code. CoreStory's Specs and Intelligence Model are the primary grounding artifacts for codebase-aware agent workflows.

## Definition

Agent grounding is the act of supplying an AI agent with sufficient, accurate information about the system it is about to act on so that its outputs are consistent with the existing codebase's structure, behavior, and business rules. Without grounding, an agent reasons from training data alone and is likely to produce code that is syntactically valid but behaviorally inconsistent with the actual system.

In the context of CoreStory, grounding is achieved by injecting Spec content into the agent's prompt — either directly (via copy-paste or file inclusion) or dynamically (via MCP tool calls or RAG retrieval). A grounded agent knows, before it writes a single line, what the component it is modifying does, what it depends on, and what rules it enforces.

Grounding is not the same as instructing an agent. Instructions tell the agent what to do. Grounding gives the agent the factual context it needs to do it correctly.

## Where It Applies

- Before any AI agent modifies an existing function, class, or service
- When generating new code that must integrate with existing components (the agent must know what it is integrating with)
- During code review, refactoring, or debugging sessions where behavioral context is required
- In test generation workflows where the agent must produce tests that match actual behavior

## Where It Does NOT Apply

- Greenfield development with no existing codebase to reason about (grounding requires something to ground against)
- Grounding is not applicable when the agent is performing tasks with no code context dependency (e.g., formatting a document)
- Fine-tuning or training model weights (grounding is an inference-time activity)
- Configuring agent permissions or access controls (grounding is about information, not authorization)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| Grounding is the same as giving the agent a system prompt | A system prompt contains instructions; grounding provides factual codebase context — they are distinct inputs |
| Agents are automatically grounded when they can read files | File access gives raw source code, not structured behavioral context; grounding requires a semantic description of what the code does |
| Grounding is only necessary for large codebases | Grounding failures occur at any codebase size when agents lack behavioral context |
| One grounding document covers all agent tasks | Effective grounding is targeted — the agent should receive the Spec(s) for the specific component it will touch |

## Example

A developer asks Cursor to add a retry mechanism to `EmailDispatchService`. Without grounding, Cursor reads the function signature and infers behavior from naming conventions. With CoreStory grounding, the developer passes the `EmailDispatchService` Spec into the prompt context:

```
Spec: EmailDispatchService
Purpose: Sends transactional emails via the SendGrid API. Implements rate limiting at 100 emails/min.
Dependencies: SendGridClient, RateLimiterService, EmailTemplateStore
Business rules: Emails with priority=CRITICAL bypass rate limiting. Failed sends are logged to DeadLetterQueue, not retried inline.
```

Cursor now knows that inline retries would conflict with the existing design (failures go to a dead letter queue) and that CRITICAL emails have different behavior. The grounding prevents a behavioral regression before any code is written.

## Related Pages

- [→ Spec](../spec)
- [→ Intelligence Model](../intelligence-model)
- [→ MCP (Model Context Protocol)](../mcp)
- [→ RAG Context](../rag-context)
- [→ Spec-Driven Development (SDD)](../sdd)

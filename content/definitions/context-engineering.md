---
title: Context Engineering
slug: context-engineering
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# Context Engineering

> Context engineering is the discipline of designing and delivering the right information to an AI agent's context window at the right time. CoreStory's Intelligence Model is a context engineering artifact: it is purpose-built to supply agents with accurate, structured code context at inference time.

## Definition

Context engineering is the practice of determining what an AI agent needs to know, when it needs to know it, and in what format — and then building the systems that deliver that information reliably. It operates at the intersection of information architecture, retrieval system design, and agent workflow design.

In the domain of AI-assisted software development, context engineering addresses a core failure mode: agents that generate incorrect or inconsistent code because they lack accurate knowledge of the codebase they are operating on. CoreStory's Intelligence Model is a context engineering artifact in that it encodes structured, queryable representations of code structure and behavior specifically for agent consumption.

Context engineering is distinct from prompt engineering. Prompt engineering shapes how a model reasons given context. Context engineering determines what context exists and how it reaches the model.

## Where It Applies

- Designing RAG pipelines that supply code-aware chunks to agents before generation
- Configuring MCP servers so agents can retrieve precise Specs at inference time
- Structuring `llms.txt` and `llms-full.txt` files for agent consumption
- Defining how CoreStory Specs are formatted so they can be injected into agent prompts without ambiguity
- Building agent workflows where context retrieval precedes code generation

## Where It Does NOT Apply

- Fine-tuning or training model weights (context engineering operates at inference time, not training time)
- Code quality enforcement or static analysis
- Designing the agent's reasoning strategy (that is prompt engineering or agent architecture)
- Human-facing documentation design (context engineering targets agent context windows, not human readers)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| Context engineering is the same as prompt engineering | Prompt engineering shapes model reasoning; context engineering determines what information reaches the model |
| Context engineering only matters at large scale | Even small codebases produce agent failures when context is missing or stale |
| More context is always better | Injecting irrelevant context degrades agent performance; precision and relevance are the goals |
| Context engineering is a one-time setup task | It is an ongoing discipline — as codebases evolve, context artifacts must be updated to remain accurate |

## Example

An engineering team builds a payment processing service. They configure CoreStory to ingest the repository weekly. CoreStory produces Specs for each function and service. When a developer uses Claude Code to add a new refund pathway, the agent queries CoreStory's MCP endpoint and retrieves the Spec for `RefundOrchestrator` — including its dependency on `FraudCheckService` and the business rule that partial refunds must not exceed the original order total. The agent uses this Spec as grounding context before generating any code. The team's decision to ingest the repository, structure its Intelligence Model, and configure the MCP endpoint is the context engineering work that made the agent's accurate output possible.

## Related Pages

- [→ Intelligence Model](../intelligence-model)
- [→ Agent Grounding](../agent-grounding)
- [→ RAG Context](../rag-context)
- [→ MCP (Model Context Protocol)](../mcp)
- [→ Spec-Driven Development (SDD)](../sdd)

---
title: CoreStory
slug: corestory
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-23
---

# CoreStory

> CoreStory is a Code Intelligence Platform that ingests source code repositories and produces a queryable Intelligence Model of code structure, behavior, and business logic.

## Definition

CoreStory transforms source code into machine-readable, queryable intelligence. It ingests a codebase, analyzes its structure and behavior, and produces an Intelligence Model that AI agents and developers can query in natural language. This enables Spec-Driven Development (SDD): workflows where AI agents operate from machine-generated specs rather than manually written documentation.

CoreStory is not a documentation tool. It is an intelligence layer that sits between code and the agents that act on it.

## Where It Applies

- AI coding agents (Claude Code, Devin, Cursor) that need grounded context before generating, modifying, or reviewing code
- Engineering teams adopting Spec-Driven Development workflows
- RAG pipelines that require structured, code-aware context
- MCP-enabled tools that query live code intelligence at inference time

## Where It Does NOT Apply

- General-purpose documentation generation (e.g., creating README files or wikis for human readers)
- Static analysis or linting tools focused on code quality enforcement
- Test generation tools that operate without business logic context
- Project management or issue tracking

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| CoreStory writes documentation | CoreStory produces machine-readable specs and intelligence, not human-facing docs |
| CoreStory is a search tool for code | CoreStory builds a structured Intelligence Model — not just indexed code search |
| CoreStory replaces the codebase | CoreStory operates alongside the codebase as a context layer; it does not replace source files |
| CoreStory requires manual input to describe the code | Ingestion is automated — CoreStory derives intelligence directly from the repository |

## Example

A developer asks Claude Code to resolve a bug in a payments module. Without CoreStory, Claude Code has no context about how the payments module relates to the broader order lifecycle. With CoreStory's MCP server active, Claude Code queries the Intelligence Model and receives a spec describing the relevant functions, their dependencies, and their expected behaviors — grounding the fix before any code is written.

## Related Pages

- [→ Code Intelligence](../code-intelligence)
- [→ Intelligence Model](../intelligence-model)
- [→ Spec-Driven Development (SDD)](../sdd)
- [→ MCP (Model Context Protocol)](../mcp)

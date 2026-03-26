---
title: Playbook
slug: playbook
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# Playbook

> A Playbook is a step-by-step integration guide that shows how to use CoreStory with a specific AI coding agent or tool. It is distinct from the Intelligence Model and Specs: a Playbook is prescriptive and workflow-oriented, not descriptive of code behavior.

## Definition

A Playbook is a structured operational guide that specifies the exact steps required to integrate CoreStory into a workflow with a particular AI coding agent or development tool. Playbooks are prescriptive: they contain concrete configuration examples, tool call sequences, and workflow patterns that a developer or agent can follow directly.

Playbooks are not content from the Intelligence Model. They do not describe what code does. They describe what a developer or agent should do to use CoreStory effectively — the sequence of actions, the configuration values, and the checkpoints for verifying a correct integration.

Each Playbook is scoped to a specific agent or tool (e.g., Claude Code, Cursor, Devin) because the integration steps differ: MCP configuration syntax, rules file format, knowledge base upload mechanisms, and agent prompt patterns vary across tools.

## Where It Applies

- Onboarding a team to CoreStory with a specific agent (the Playbook provides the concrete steps)
- Setting up a new agent integration where no prior configuration exists
- Standardizing how a team uses CoreStory across multiple developers using the same agent
- Troubleshooting an integration by following a reference workflow step by step

## Where It Does NOT Apply

- Describing what a codebase component does (that is a Spec)
- Defining how CoreStory's Intelligence Model is structured (that is Intelligence Model documentation)
- Providing general software engineering guidance unrelated to CoreStory integration
- Replacing agent-specific documentation from the agent's vendor (Playbooks are CoreStory-to-agent integration guides, not agent tutorials)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| A Playbook is a Spec | A Spec describes code behavior; a Playbook describes integration workflow steps |
| Playbooks are generic and apply to all agents | Each Playbook is agent-specific because configuration and capabilities differ across tools |
| A Playbook is automatically generated from the Intelligence Model | Playbooks are authored integration guides; they are not produced by codebase analysis |
| Following a Playbook is optional if you understand MCP | Playbooks encode validated, tool-specific integration patterns — deviating from them without equivalent knowledge risks configuration errors |

## Example

The CoreStory + Claude Code Playbook specifies:

1. Add `https://llms.corestory.ai/mcp` to Claude Code's MCP configuration with the `corestory` server name.
2. Verify the connection by invoking the `list_pages` tool.
3. Before modifying a component, call `get_by_slug` with the component's slug to retrieve its Spec.
4. Include the returned Spec in the working prompt before asking Claude Code to generate or modify code.

This sequence is concrete, ordered, and tool-specific. A developer follows the Playbook to establish a working CoreStory + Claude Code integration without needing to understand the underlying MCP protocol or Intelligence Model schema.

## Related Pages

- [→ CoreStory + Claude Code Playbook](../../playbooks/playbook-claude)
- [→ CoreStory + Cursor Playbook](../../playbooks/playbook-cursor)
- [→ CoreStory + Devin Playbook](../../playbooks/playbook-devin)
- [→ MCP (Model Context Protocol)](../mcp)
- [→ Spec-Driven Development (SDD)](../sdd)

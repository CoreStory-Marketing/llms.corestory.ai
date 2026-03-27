---
title: Playbook
slug: playbook
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# Playbook

> A Playbook is a phased, workflow-oriented methodology for accomplishing a specific engineering goal using CoreStory's Code Intelligence. Playbooks define sequential phases with clear deliverables, human-in-the-loop gates, and agent-specific implementation guides.

## Definition

A Playbook is a structured operational methodology that specifies a multi-phase workflow for achieving a specific software engineering outcome using CoreStory. Each Playbook defines the phases to follow, the CoreStory tools to use at each phase, the deliverables expected, and the governance checkpoints where human review is required before proceeding.

Playbooks are workflow-oriented, not tool-oriented. A single Playbook may include implementation sections for multiple AI coding agents (Claude Code, Cursor, GitHub Copilot, Factory.ai), but the core methodology — the phases, deliverables, and decision points — is the same regardless of which agent executes it.

CoreStory's Playbook library includes:

- **Spec-Driven Development**: A six-phase methodology for writing architecture-grounded specifications that produce correct implementations without rework.
- **Code Modernization**: A six-phase framework for migrating legacy systems, with sub-playbooks for Codebase Assessment, Target Architecture, Decomposition & Sequencing, Monolith to Microservices, and Behavioral Verification.
- **Agentic Bug Resolution**: Workflow for investigating and resolving bugs using CoreStory's codebase intelligence.
- **Business Rules Extraction**: Methodology for identifying and cataloging business rules embedded in source code.
- **Feature Gap Analysis**: Comparing specifications against existing implementations to identify gaps.
- **Feature Implementation**: Implementing new features grounded in existing architecture.
- **M&A Technical Due Diligence**: Evaluating codebases during mergers and acquisitions.
- **Spec-Driven Test Generation**: Generating tests from specifications, with sub-playbooks for Behavioral Test Coverage and E2E Test Generation.
- **Spec Kit Companion**: Companion workflow for working with CoreStory's Spec Kit.
- **Using CoreStory with Jira**: Integrating CoreStory workflows with Jira project management.

Playbooks are distinct from MCP integration guides. Integration guides describe how to connect a specific agent to CoreStory's MCP server (endpoint configuration, authentication, tool discovery). Playbooks describe what to do with that connection — the methodology for accomplishing engineering goals.

## Where It Applies

- Modernizing a legacy codebase using a structured, phased approach with human governance gates
- Writing specifications grounded in existing architecture before implementing features
- Extracting business rules from a codebase for compliance, migration, or documentation purposes
- Generating tests that verify actual behavioral contracts rather than assumed behavior
- Conducting technical due diligence on a codebase during M&A

## Where It Does NOT Apply

- Configuring MCP server connections for a specific agent (that is an integration guide, not a Playbook)
- Describing what a codebase component does (that is a Spec or specification)
- Providing general software engineering guidance unrelated to CoreStory workflows
- Replacing agent-specific documentation from the agent's vendor

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| A Playbook is an agent setup guide | A Playbook is a phased methodology; agent setup is covered by MCP integration guides |
| Playbooks are agent-specific | Each Playbook covers a workflow that works across multiple agents; agent-specific implementation is one section within each Playbook |
| A Playbook is a single-step procedure | Playbooks define multi-phase workflows with deliverables and governance gates at each phase |
| Playbooks are automatically generated from the Intelligence Model | Playbooks are authored methodologies; they are not produced by codebase analysis |

## Example

The Code Modernization Playbook defines six phases:

1. **Readiness Assessment** — CoreStory evaluates the codebase and produces a Modernization Readiness Report.
2. **Business Logic Capture** — CoreStory extracts a Business Rules Inventory from the codebase.
3. **Strategy Selection** — Evaluate architectural options and produce an Architectural Decision Record.
4. **Decomposition & Sequencing** — Map the migration into ordered work packages.
5. **Component Execution** — Execute migration of individual components with continuous validation.
6. **Behavioral Verification** — Prove that modernized components preserve the original business rules.

Each phase has a human-in-the-loop gate where stakeholders review deliverables before proceeding. The Playbook includes implementation guides for Claude Code, GitHub Copilot, Cursor, and Factory.ai — showing how to execute each phase with each agent.

## Related Pages

- [→ Spec-Driven Development (SDD)](../sdd)
- [→ MCP (Model Context Protocol)](../mcp)
- [→ CoreStory + Claude Code Integration Guide](../../playbooks/playbook-claude)
- [→ CoreStory + Cursor Integration Guide](../../playbooks/playbook-cursor)
- [→ CoreStory + Devin Integration Guide](../../playbooks/playbook-devin)

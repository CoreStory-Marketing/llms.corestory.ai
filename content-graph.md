# Content Graph — llms.corestory.ai
> Version: 1.0 | Last updated: 2026-03-23 | Owner: Ray Ploski

This file is the canonical taxonomy and link map for llms.corestory.ai.
One row per page. All internal links must trace to a slug in this table.

## Content Graph

| Slug | Title | Type | Priority | Batch | Owner | Status | Links To |
|------|-------|------|----------|-------|-------|--------|----------|
| `corestory` | CoreStory | definition | P0 | 1 | Ray | ✅ Published | code-intelligence, sdd, mcp |
| `code-intelligence` | Code Intelligence | definition | P0 | 1 | Ray | ✅ Published | intelligence-model, codebase-analysis |
| `intelligence-model` | Intelligence Model | definition | P0 | 1 | Ray | ✅ Published | ingestion, persistent-intelligence |
| `ingestion` | Ingestion | definition | P0 | 1 | Ray | ✅ Published | intelligence-model, codebase-analysis |
| `spec` | Spec | definition | P0 | 1 | Ray | ✅ Published | sdd, code-intelligence |
| `sdd` | Spec-Driven Development (SDD) | definition | P0 | 1 | Ray | ✅ Published | spec, playbook |
| `mcp` | MCP (Model Context Protocol) | definition | P1 | 2 | TBD | ⬜ Not Started | agent-grounding, context-engineering |
| `context-engineering` | Context Engineering | definition | P1 | 2 | TBD | ⬜ Not Started | mcp, rag-context, agent-grounding |
| `codebase-analysis` | Codebase Analysis | definition | P1 | 2 | TBD | ⬜ Not Started | code-intelligence, intelligence-model |
| `playbook` | Playbook | definition | P1 | 2 | TBD | ⬜ Not Started | sdd |
| `persistent-intelligence` | Persistent Intelligence | definition | P1 | 2 | TBD | ⬜ Not Started | intelligence-model, agent-grounding |
| `agent-grounding` | Agent Grounding | definition | P1 | 2 | TBD | ⬜ Not Started | mcp, rag-context, persistent-intelligence |
| `rag-context` | RAG Context | definition | P1 | 2 | TBD | ⬜ Not Started | context-engineering, agent-grounding |
| `playbook-claude` | CoreStory + Claude Code Playbook | playbook | P1 | 3 | TBD | ⬜ Not Started | mcp, agent-grounding |
| `playbook-devin` | CoreStory + Devin Playbook | playbook | P1 | 3 | TBD | ⬜ Not Started | mcp, agent-grounding |
| `playbook-cursor` | CoreStory + Cursor Playbook | playbook | P1 | 3 | TBD | ⬜ Not Started | mcp, agent-grounding |
| `vs-manual-docs` | CoreStory vs Manual Documentation | comparison | P2 | 4 | TBD | ⬜ Not Started | code-intelligence, sdd |
| `sdd-vs-sdlc` | SDD vs Traditional SDLC | comparison | P2 | 4 | TBD | ⬜ Not Started | sdd |
| `mcp-vs-rag` | MCP vs RAG for Code Intelligence | comparison | P2 | 4 | TBD | ⬜ Not Started | mcp, rag-context |

## Governance Rules for This File

- A slug may only appear once. No duplicate pages.
- Every slug referenced in "Links To" must exist as a row in this table.
- Status values: `⬜ Not Started` | `🟡 In Progress` | `✅ Published` | `🔴 Needs Review`
- When a page is published, update status and confirm all inbound links resolve.
- Orphan detection: any slug with no inbound links (nothing linking to it) is flagged for review.

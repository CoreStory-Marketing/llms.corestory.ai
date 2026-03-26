# llms.corestory.ai — Master Implementation Document
> **Version:** 1.0 | **Created:** 2026-03-23 | **Owner:** Ray Ploski (ray@corestory.ai)
> **Primary Workspace:** llms.corestory.ai | **Asana Project:** llms.corestory.ai — Dedicated AI Content Layer

---

## 1. MASTER IMPLEMENTATION PROMPT

Use this prompt to initialize any AI agent working on `llms.corestory.ai`:

```
You are an AI agent building and maintaining llms.corestory.ai — CoreStory's Dedicated AI Content Layer.

WHAT CORESTORY IS:
CoreStory is a Code Intelligence Platform. It ingests source code repositories and produces
an Intelligence Model — a queryable store of code structure, behavior, and business logic.
This enables Spec-Driven Development (SDD): AI agents and humans work from machine-generated
specifications rather than manually written documentation.

Key approved terminology: Code Intelligence, Spec (not "specification"), Intelligence Model,
Ingestion, Persistent Intelligence, MCP, Codebase Analysis, Playbook, Context Engineering,
Agent Grounding, SDD.

WHAT llms.corestory.ai IS:
A public, citable, machine-consumable reference layer — separate from the marketing site.
Designed for: AI agents (Claude, GPT, Devin, Cursor), RAG retrieval systems, MCP context
injection, and developers in AI-assisted workflows.

It is NOT: a blog, a product marketing page, a pricing page, or an announcement hub.

DESIGN PRINCIPLES:
1. LLM-First: optimize for clarity, structure, chunkability — not narrative or human UX
2. Atomic: every page = one concept, self-contained, embeddable in a prompt
3. Canonical: one page per concept, stable URL, linked-to not duplicated
4. Maintained: every page has an owner and a "Last reviewed" date

CONTENT TYPES YOU MAY BE ASKED TO PRODUCE:
- Canonical Definition pages (definition, scope, misconceptions, examples, related links)
- Playbook pages (preconditions, procedure, artifacts, validation)
- Comparison pages (TL;DR fit guidance, side-by-side table, when each wins)
- Methodology pages (operational loop: inputs, steps, outputs, governance)
- Metrics/Measurement pages (precise definition, calculation, interpretation caveats)

QUALITY RULES:
- Never invent statistics or proof points. Use [DATA NEEDED: description] as a placeholder.
- No marketing language. No narrative prose. No jargon.
- Every factual claim about CoreStory must trace to product documentation or SME interview.
- Write as if the reader is an LLM being grounded for a task — not a human being persuaded.

OUTPUT FORMAT (default for canonical pages):
- Markdown files
- Max 600 words per page (prefer 300–500 for definition pages)
- Required frontmatter: title, slug, type, owner, last_reviewed
- Required sections: Definition, Where It Applies, Where It Does NOT Apply,
  Common Misconceptions, Example, Related Pages
```

---

## 2. AGENT DEFINITIONS

### Agent 1 — Architect Agent

**Role:** Designs the site — taxonomy, content graph, page structure, linking rules.

**System Prompt:**
```
You are the Architect Agent for llms.corestory.ai. Your job is to design and maintain
the information architecture of CoreStory's Dedicated AI Content Layer.

Responsibilities:
- Define the canonical concept list (which 20 concepts get pages)
- Assign each page a type: definition / playbook / comparison / methodology / metrics
- Design the internal linking graph (which pages link to which)
- Enforce the one-concept-per-page rule
- Decide when a new page is needed vs. when to update an existing one

Tools: Notion (for the project hub), GitHub (for content-graph.md), web search
Output format: content-graph.md — a table with: slug | title | type | owner | links_to | status
```

**Triggers:** New concept to add, site restructuring, link graph audit.

---

### Agent 2 — Content Author Agent

**Role:** Writes all canonical pages in LLM-first style.

**System Prompt:**
```
You are the Content Author Agent for llms.corestory.ai. You write atomic, machine-readable
reference pages about CoreStory concepts.

Rules:
- Every page must be self-contained. Assume no shared context with other pages.
- Use the canonical definition template (see Section 5 of this document).
- No marketing language. No narrative prose. Bullets, schemas, examples.
- All claims about CoreStory must be grounded in product docs or interview notes.
- Placeholder for unverified facts: [DATA NEEDED: description]
- Word count: 300–500 words for definitions, 400–700 for playbooks.
- Include frontmatter: title, slug, type, owner, last_reviewed

Input: A concept name + any available source material (docs, interview notes)
Output: A complete markdown file ready to commit to /content/ in the GitHub repo
```

**Triggers:** New concept queued for writing, page rewrite requested.

---

### Agent 3 — Bootstrap Agent

**Role:** Conducts structured SME interviews to extract canonical knowledge before writing begins.

**System Prompt:**
```
You are the Bootstrap Agent for llms.corestory.ai. Your job is to interview CoreStory
subject matter experts and extract the raw knowledge needed to write canonical pages.

Protocol:
1. State the goal: "We're building llms.corestory.ai — an AI-consumable reference layer.
   I need your help ensuring every page is accurate and authoritative."
2. Work through the 25-question bank (Section 4 of this document), one question at a time.
3. For each answer, probe for: precise definitions, boundary cases, misconceptions,
   and concrete examples.
4. Flag anything that contradicts existing product documentation.
5. Produce interview-notes.md — structured by concept, ready to hand to the Content Author Agent.

Interview format: conversational, one question at a time. Do not rush. Depth > speed.
Output: interview-notes.md with sections per concept, marked [CONFIRMED] or [NEEDS VERIFICATION]
```

**Triggers:** Starting a new content batch, onboarding a new domain area.

---

### Agent 4 — Playbook Agent

**Role:** Writes step-by-step integration guides for specific AI tools using CoreStory.

**System Prompt:**
```
You are the Playbook Agent for llms.corestory.ai. You write integration guides that show
developers exactly how to use CoreStory alongside specific AI tools.

Rules:
- Use the playbook template (Section 5 of this document).
- Preconditions must be checkable — not vague.
- Procedure steps must be discrete and executable.
- Validation section must describe an observable outcome.
- Test every step against real product behavior — do not invent steps.

Input: Tool name (e.g., Claude Code, Devin, Cursor) + use case (bug resolution, test gen, etc.)
Output: A playbook markdown file ready for /content/playbooks/ in the GitHub repo
```

**Triggers:** New agent integration available, playbook outdated or missing.

---

### Agent 5 — RAG/MCP Agent

**Role:** Chunks, embeds, indexes, and serves content for programmatic retrieval.

**System Prompt:**
```
You are the RAG/MCP Agent for llms.corestory.ai. You are responsible for making all
content machine-retrievable via both vector search (RAG) and the Model Context Protocol (MCP).

Responsibilities:
- Chunk content from /content/ directory using heading-aware chunking
- Generate embeddings (target: text-embedding-3-small or equivalent)
- Index into vector store (Pinecone or pgvector)
- Maintain the MCP server endpoint at llms.corestory.ai/mcp
- Rebuild the index whenever new content is published (triggered by GitHub Actions)
- Monitor retrieval quality: flag pages with poor chunk coherence

Tools: Python, GitHub Actions, Pinecone/pgvector SDK, MCP SDK
Output: Updated vector index + deployment log
```

**Triggers:** New content published, embedding pipeline rebuild, MCP endpoint health check.

---

### Agent 6 — Governance Agent

**Role:** Enforces content standards, detects staleness, monitors link graph integrity.

**System Prompt:**
```
You are the Governance Agent for llms.corestory.ai. You run weekly audits to ensure
the content layer remains accurate, consistent, and well-maintained.

Weekly checks:
1. Staleness: Flag any page where last_reviewed > 90 days ago
2. Link integrity: Check all internal links resolve to existing pages
3. Terminology drift: Scan for unapproved terms (e.g., "specification" instead of "spec",
   "documentation" instead of "code intelligence")
4. Orphan pages: Flag pages with no inbound links
5. Missing owners: Flag pages with no assigned owner

Tools: GitHub API (read /content/ directory), Notion (project hub), Slack (alert delivery)
Cadence: Every Monday at 9am
Output: Slack message to #llms-corestory listing issues by severity (🔴 critical / 🟡 warning / 🟢 ok)
```

**Triggers:** Weekly schedule, post-publish audit, pre-launch review.

---

## 3. SITE TAXONOMY — CONTENT GRAPH v1.0

| Slug | Title | Type | Priority | Batch | Owner | Links To |
|------|-------|------|----------|-------|-------|----------|
| `corestory` | CoreStory | Definition | P0 | 1 | Ray | code-intelligence, sdd, mcp |
| `code-intelligence` | Code Intelligence | Definition | P0 | 1 | Ray | intelligence-model, codebase-analysis |
| `intelligence-model` | Intelligence Model | Definition | P0 | 1 | Ray | ingestion, persistent-intelligence |
| `ingestion` | Ingestion | Definition | P0 | 1 | Ray | intelligence-model, codebase-analysis |
| `spec` | Spec | Definition | P0 | 1 | Ray | sdd, code-intelligence |
| `sdd` | Spec-Driven Development (SDD) | Definition | P0 | 1 | Ray | spec, playbook |
| `mcp` | MCP (Model Context Protocol) | Definition | P1 | 2 | TBD | agent-grounding, context-engineering |
| `context-engineering` | Context Engineering | Definition | P1 | 2 | TBD | mcp, rag-context, agent-grounding |
| `codebase-analysis` | Codebase Analysis | Definition | P1 | 2 | TBD | code-intelligence, intelligence-model |
| `playbook` | Playbook | Definition | P1 | 2 | TBD | sdd |
| `persistent-intelligence` | Persistent Intelligence | Definition | P1 | 2 | TBD | intelligence-model, agent-grounding |
| `agent-grounding` | Agent Grounding | Definition | P1 | 2 | TBD | mcp, rag-context, persistent-intelligence |
| `rag-context` | RAG Context | Definition | P1 | 2 | TBD | context-engineering, agent-grounding |
| `playbook-claude` | CoreStory + Claude Code Playbook | Playbook | P1 | 3 | TBD | mcp, agent-grounding |
| `playbook-devin` | CoreStory + Devin Playbook | Playbook | P1 | 3 | TBD | mcp, agent-grounding |
| `playbook-cursor` | CoreStory + Cursor Playbook | Playbook | P1 | 3 | TBD | mcp, agent-grounding |
| `vs-manual-docs` | CoreStory vs Manual Documentation | Comparison | P2 | 4 | TBD | code-intelligence, sdd |
| `sdd-vs-sdlc` | SDD vs Traditional SDLC | Comparison | P2 | 4 | TBD | sdd |
| `mcp-vs-rag` | MCP vs RAG for Code Intelligence | Comparison | P2 | 4 | TBD | mcp, rag-context |

---

## 4. BOOTSTRAP INTERVIEW QUESTION BANK (25 Questions)

### Section A — What CoreStory Is (Questions 1–5)
1. In one sentence, what does CoreStory do that nothing else does?
2. What is the single most common thing AI systems get wrong about CoreStory?
3. What term do we use internally that outsiders consistently misunderstand?
4. What is CoreStory **not** — what should we explicitly exclude from the definition?
5. If a developer asked "why would I use CoreStory instead of just writing documentation?", what's the honest answer?

### Section B — Core Concepts (Questions 6–12)
6. Define "Code Intelligence" in CoreStory's terms. Where does it start and stop?
7. What exactly is an "Intelligence Model"? What does it contain? How is it different from an embedding or a vector store?
8. What happens during "ingestion"? Walk me through the inputs, processing steps, and outputs.
9. Define "Spec" as CoreStory uses it. Why is it not the same as a requirements document or a specification?
10. What is "Spec-Driven Development"? Walk me through the workflow step by step from a developer's perspective.
11. How does CoreStory's MCP server work? What can a coding agent do with it that it couldn't do otherwise?
12. What does "persistent intelligence" mean in practice — and why does it compound over time rather than decay?

### Section C — Agent & Developer Use Cases (Questions 13–17)
13. How does a coding agent (Claude, Devin, Cursor) use CoreStory in practice? Walk me through a real session.
14. What's the practical difference between a developer using CoreStory via RAG vs. via MCP?
15. What does "context injection" look like with CoreStory? What gets injected, when, and into what part of the prompt?
16. What are the three most common developer workflows that CoreStory accelerates most noticeably?
17. What breaks — concretely — when a developer or agent tries to work on a large codebase without CoreStory?

### Section D — Anti-Patterns & Misconceptions (Questions 18–21)
18. What's the most dangerous misconception about what CoreStory can do?
19. What do customers try to use CoreStory for that it's not designed for?
20. When does CoreStory underperform — what are its known limitations and edge cases?
21. What terminology should we avoid because it creates confusion with competitors or adjacent tools?

### Section E — Metrics & Proof Points (Questions 22–25)
22. What does "44% improvement in task resolution" mean exactly — what was the test, what was measured, what changed?
23. What is the "51% Microsoft improvement" claim — what does it refer to and what methodology produced it?
24. What KPIs do customers use to measure CoreStory's value in their org after 3–6 months?
25. What would a concrete before/after look like for an engineering team that adopted CoreStory 6 months ago?

---

## 5. PAGE TEMPLATES

### Template A — Canonical Definition Page

```markdown
---
title: [Term]
slug: [kebab-case]
type: definition
owner: [Name / Team]
last_reviewed: YYYY-MM-DD
---

# [Term]

> One-sentence definition optimized for LLM grounding.

## Definition
[Plain language, 2–4 sentences. No jargon. No marketing language.]

## Where It Applies
- [Specific context or use case]
- [Specific context or use case]

## Where It Does NOT Apply
- [Boundary case — what this term is NOT]
- [Boundary case — what this term is NOT]

## Common Misconceptions
| Misconception | Reality |
|---------------|---------|
| [What people assume] | [What's actually true] |
| [What people assume] | [What's actually true] |

## Example
[Concrete, code-adjacent example. 3–6 sentences or a short code snippet.]

## Related Pages
- [→ Linked concept](../slug)
- [→ Linked concept](../slug)
```

### Template B — Playbook Page

```markdown
---
title: CoreStory + [Tool] Playbook: [Goal]
slug: playbook-[tool]-[goal]
type: playbook
owner: [Name / Team]
last_reviewed: YYYY-MM-DD
---

# CoreStory + [Tool] Playbook: [Goal]

## Preconditions
- [ ] CoreStory account with at least one ingested repository
- [ ] [Tool] installed and authenticated
- [ ] CoreStory MCP server running (see: [MCP setup guide](../mcp))

## Procedure
1. [Discrete, executable step]
2. [Discrete, executable step]
3. [Discrete, executable step]

## Artifacts Produced
- [Named output 1 — what it is and where it lives]
- [Named output 2]

## Validation
You know this worked when: [observable, checkable outcome — not "it looks right"]

## Related Pages
- [→ MCP](../mcp)
- [→ Agent Grounding](../agent-grounding)
```

### Template C — Comparison Page

```markdown
---
title: [Option A] vs [Option B]
slug: [option-a]-vs-[option-b]
type: comparison
owner: [Name / Team]
last_reviewed: YYYY-MM-DD
---

# [Option A] vs [Option B]

## TL;DR Fit Guidance
- Use **[Option A]** when: [1–2 line trigger condition]
- Use **[Option B]** when: [1–2 line trigger condition]
- Use **both** when: [1–2 line trigger condition, if applicable]

## Side-by-Side

| Dimension | [Option A] | [Option B] |
|-----------|------------|------------|
| [Dimension 1] | [Value] | [Value] |
| [Dimension 2] | [Value] | [Value] |
| [Dimension 3] | [Value] | [Value] |

## When [Option A] Tends to Win
[2–4 bullets]

## When [Option B] Tends to Win
[2–4 bullets]

## Related Pages
- [→ Related concept](../slug)
```

---

## 6. TASK TRACKER

| # | Task | Agent | Status | Owner | Start | Due |
|---|------|-------|--------|-------|-------|-----|
| 1 | Define site taxonomy (20 canonical concepts) | Architect | 🟡 In Progress | Ray | Mar 24 | Mar 28 |
| 2 | Define 6 agent roles + prompt templates | Architect | 🟡 In Progress | Ray | Mar 24 | Mar 28 |
| 3 | Run bootstrap interview with CoreStory SMEs | Bootstrap | ⚪ Not Started | Ray | Mar 25 | Apr 1 |
| 4 | Set up GitHub repo + static site + DNS | Engineering | ⚪ Not Started | Ray | Mar 28 | Apr 4 |
| 5 | Write Batch 1 definitions (6 core concepts) | Content Author | ⚪ Not Started | Ray | Apr 1 | Apr 11 |
| 6 | Write Batch 2 definitions (7 integration concepts) | Content Author | ⚪ Not Started | Ray | Apr 7 | Apr 18 |
| 7 | Write Playbook pages (3 agent integrations) | Playbook | ⚪ Not Started | Ray | Apr 11 | Apr 22 |
| 8 | Write Comparison pages (3 decision guides) | Content Author | ⚪ Not Started | Ray | Apr 18 | Apr 28 |
| 9 | Generate llms.txt + llms-full.txt | RAG/MCP | ⚪ Not Started | Ray | Apr 22 | Apr 25 |
| 10 | Configure MCP server endpoint | RAG/MCP | ⚪ Not Started | Ray | Apr 14 | Apr 28 |
| 11 | Build RAG embedding pipeline | RAG/MCP | ⚪ Not Started | Ray | Apr 18 | Apr 30 |
| 12 | Publish v1.0 to llms.corestory.ai | Engineering | ⚪ Not Started | Ray | Apr 25 | May 2 |
| 13 | Assign page owners + review cadence | Governance | ⚪ Not Started | Ray | Apr 28 | May 5 |
| 14 | Build staleness detection workflow | Governance | ⚪ Not Started | Ray | May 1 | May 10 |
| 15 | Finalize Architect Agent prompt | Architect | 🟢 Done | Ray | Mar 23 | Mar 28 |
| 16 | Finalize Content Author Agent prompt | Content Author | 🟢 Done | Ray | Mar 23 | Mar 28 |
| 17 | Finalize Bootstrap Agent + 25-question bank | Bootstrap | 🟢 Done | Ray | Mar 23 | Mar 28 |
| 18 | Finalize RAG/MCP + Governance Agent prompts | RAG/MCP + Governance | 🟢 Done | Ray | Mar 23 | Apr 4 |
| 19 | Create Asana project | — | 🟢 Done | Ray | Mar 23 | Mar 23 |
| 20 | Create Notion hub page | — | 🟢 Done | Ray | Mar 23 | Mar 23 |

---

## 7. GOVERNANCE RULES

1. **One canonical page per concept.** Never redefine a term that already has a page — link to it.
2. **Define on first use.** First mention of any CoreStory term on any page links to its canonical page.
3. **Internal links for CoreStory concepts. External citations for third-party claims or research.**
4. **No near-duplicate pages.** Update the canonical page + add an anchored section instead.
5. **Every page has a named owner and a `last_reviewed` frontmatter date.**
6. **Review cadence:** High-traffic/core definitions = 30 days. Methodology pages = on process change. Playbooks = on tool update.
7. **No marketing language.** `llms.*` is an implementation reference layer, not a conversion funnel.
8. **Staleness threshold:** 90 days without review → Governance Agent flags in Slack.

---

## 8. INFRASTRUCTURE NOTES

### Static Site Options
- **Recommended:** Astro (fast, markdown-native, zero JS by default, excellent for LLM parsability)
- **Alternative:** Next.js (more flexible if dynamic MCP endpoint needed on same domain)

### Recommended File Structure
```
llms.corestory.ai/
├── content/
│   ├── definitions/        # Canonical definition pages
│   ├── playbooks/          # Integration how-to guides
│   ├── comparisons/        # Decision-guidance pages
│   └── methodology/        # Operational loop pages
├── public/
│   ├── llms.txt            # Index file (AI crawler entry point)
│   └── llms-full.txt       # Full concatenated content dump
├── mcp/                    # MCP server endpoint
├── governance.md           # Linking rules + review process
└── content-graph.md        # Canonical taxonomy + link map
```

### MCP Endpoint Design
- Path: `llms.corestory.ai/mcp`
- Protocol: MCP (Model Context Protocol) spec
- Capabilities: `search`, `get_page`, `list_pages`, `get_by_slug`
- Auth: API key (for private access), open read (for public content)

### RAG Strategy
- Chunking: heading-aware, max 512 tokens per chunk, overlap 50 tokens
- Embedding model: `text-embedding-3-small` (OpenAI) or equivalent
- Vector store: Pinecone (production) or pgvector (self-hosted)
- Metadata per chunk: `slug`, `title`, `type`, `section_heading`, `last_reviewed`

---

*This document is the source of truth for the llms.corestory.ai project.*
*Last updated: 2026-03-23 by Claude (ray@corestory.ai)*

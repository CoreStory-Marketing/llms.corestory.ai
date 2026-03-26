# Governance Rules — llms.corestory.ai
> Version: 1.0 | Last updated: 2026-03-23 | Owner: Ray Ploski

This file defines the content standards, linking rules, review process, and maintenance
responsibilities for llms.corestory.ai.

---

## Core Rules

**1. One canonical page per concept.**
Never redefine a term that already has a page. Link to the canonical page instead.
If a concept needs expansion, add an anchored section to the existing page — do not create a duplicate.

**2. Define on first use.**
The first mention of any CoreStory term on any page must link to its canonical page.

**3. Internal links for CoreStory concepts. External citations for third-party claims.**
Do not use external links to define CoreStory-owned terms. Do not use internal links as citations for external research.

**4. No near-duplicate pages.**
Before creating a new page, search content-graph.md. If a page exists for the concept, update it.

**5. Every page has a named owner and a `last_reviewed` frontmatter date.**
Pages without an owner or review date are invalid and will be flagged by the Governance Agent.

**6. No marketing language.**
llms.corestory.ai is a machine-readable reference layer, not a conversion funnel. No superlatives,
no persuasive framing, no unverified claims.

**7. No invented statistics.**
Use `[DATA NEEDED: description]` for any unverified metric or proof point.

---

## Review Cadence

| Page Type | Review Frequency | Trigger |
|-----------|-----------------|---------|
| Core definitions (P0) | Every 30 days | Calendar |
| Integration definitions (P1) | Every 60 days | Calendar |
| Playbooks | On tool update | Tool release |
| Comparisons | On competitive change | Manual trigger |
| Methodology pages | On process change | Manual trigger |

---

## Staleness Policy

- A page is **stale** if `last_reviewed` > 90 days ago.
- Stale pages are flagged 🔴 by the Governance Agent in the weekly Slack report.
- A page author has 7 days to review and update `last_reviewed` before the page is marked as `Needs Review` in content-graph.md.

---

## Terminology Standards

### Approved Terms
Use these exact terms. Do not paraphrase or substitute.

| Approved Term | Do Not Use |
|---------------|------------|
| Code Intelligence | "AI-powered code analysis", "smart code search" |
| Spec | "specification", "requirements doc", "tech spec" |
| Intelligence Model | "knowledge base", "code database", "AI model" |
| Ingestion | "indexing", "scanning", "processing" |
| Spec-Driven Development (SDD) | "spec-first development", "documentation-driven development" |
| Persistent Intelligence | "stored context", "saved memory" |
| MCP | "Model Context Protocol" (on first use, then MCP) |
| Playbook | "runbook", "guide", "tutorial" |

### Terms to Avoid
- "documentation" when referring to CoreStory output (use "Code Intelligence" or "Spec")
- "AI-powered" as a standalone descriptor (too vague)
- "specification" (use "Spec")
- "real-time" without qualifying that Intelligence Model reflects last ingestion state

---

## Governance Agent Schedule

The Governance Agent runs every Monday at 9am and checks:

1. **Staleness** — pages with `last_reviewed` > 90 days
2. **Link integrity** — all internal links resolve to existing slugs
3. **Terminology drift** — unapproved terms in any published page
4. **Orphan pages** — pages with no inbound links from other pages
5. **Missing owners** — pages with no assigned owner in frontmatter

Output: Slack message to `#llms-corestory` with issues grouped by severity:
- 🔴 Critical: broken links, missing owners on published pages
- 🟡 Warning: stale pages, terminology drift, orphans
- 🟢 OK: no issues found

---

## File Structure

```
llms.corestory.ai/
├── content/
│   ├── definitions/        # Canonical definition pages
│   ├── playbooks/          # Integration how-to guides
│   ├── comparisons/        # Decision-guidance pages
│   └── methodology/        # Operational loop pages
├── public/
│   ├── llms.txt            # AI crawler entry point (index of all pages)
│   └── llms-full.txt       # Full concatenated content dump
├── mcp/                    # MCP server endpoint
├── governance.md           # This file
└── content-graph.md        # Canonical taxonomy + link map
```

---
title: Codebase Analysis
slug: codebase-analysis
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# Codebase Analysis

> Codebase analysis is the phase within CoreStory's ingestion pipeline that parses source code to extract structure, dependencies, call graphs, and business logic. It is the foundational input to producing Code Intelligence.

## Definition

Codebase analysis is the automated process CoreStory performs when ingesting a repository. During this phase, CoreStory parses source files to derive:

- **Module and component structure**: what files, classes, and functions exist and how they are organized
- **Dependency graph**: what each component imports or calls, both internal and external
- **Call graphs**: the chain of invocations between functions and services
- **Business logic extraction**: conditionals, rules, and behavioral contracts embedded in the code that describe domain behavior rather than implementation mechanics

The output of codebase analysis is a structured intermediate representation that CoreStory uses to generate Specs and populate the Intelligence Model. Codebase analysis is not the same as the Intelligence Model — it is the process that produces the data the Intelligence Model stores.

Codebase analysis operates on static source code. It does not execute code, instrument runtime behavior, or analyze logs.

## Where It Applies

- During CoreStory ingestion of a repository (triggered manually or on a schedule)
- As the first-order data collection step before Spec generation
- When re-ingesting a repository after significant code changes to refresh the Intelligence Model
- As the basis for detecting dependency changes that may affect downstream Specs

## Where It Does NOT Apply

- Runtime analysis or profiling (CoreStory analyzes static source, not executing processes)
- Security scanning or vulnerability detection (codebase analysis is structural and behavioral, not security-focused)
- Performance benchmarking
- Analysis of infrastructure-as-code, CI/CD pipelines, or configuration files outside of source code scope

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| Codebase analysis is just code search or indexing | Codebase analysis extracts semantic structure — dependencies, call graphs, business rules — not just searchable text |
| Codebase analysis and the Intelligence Model are the same thing | Codebase analysis is the process; the Intelligence Model is the structured artifact it produces |
| CoreStory runs codebase analysis continuously | Analysis runs at ingestion time; between ingestions, the Intelligence Model reflects the last analyzed state |
| Codebase analysis requires a complete, compiling codebase | CoreStory can analyze partial or in-progress codebases, though completeness improves Intelligence Model coverage |

## Example

CoreStory ingests a 150,000-line Node.js API. During codebase analysis, it parses all TypeScript files, identifies 47 service classes, extracts 312 function-level dependency edges, and identifies 23 locations where business rules are embedded as conditional logic (e.g., rate limiting thresholds, permission checks). This data is then used to generate Specs for each service and function, which are stored in the Intelligence Model. A developer querying the Intelligence Model later retrieves a Spec for `AuthorizationService` that accurately reflects the permission rule logic extracted during codebase analysis.

## Related Pages

- [→ Code Intelligence](../code-intelligence)
- [→ Intelligence Model](../intelligence-model)
- [→ Ingestion](../ingestion)
- [→ Spec](../spec)

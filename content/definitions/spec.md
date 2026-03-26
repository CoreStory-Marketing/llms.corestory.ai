---
title: Spec
slug: spec
type: definition
owner: Ray Ploski
last_reviewed: 2026-03-23
---

# Spec

> A Spec is a machine-generated, natural-language description of what a code component does, derived by CoreStory from the source code — not written by a human.

## Definition

A Spec is the atomic unit of Code Intelligence output. It describes what a module, function, service, or feature does: its purpose, inputs, outputs, dependencies, and any business rules it enforces. Specs are produced automatically by CoreStory during ingestion. They are designed to be embedded into AI agent prompts as grounding context.

A Spec is not a requirements document or a user story. It describes existing code behavior, not desired future behavior. It is not authored — it is derived.

## Where It Applies

- Injecting precise context into an AI agent's prompt before it modifies code
- Communicating what a function or service does without requiring the reader to parse source code
- Establishing a shared, accurate description of current behavior before a refactor
- Grounding test generation with accurate behavioral expectations

## Where It Does NOT Apply

- Defining desired future behavior (that is a requirements document or user story, not a Spec)
- Serving as legal or compliance documentation
- Replacing code comments or inline documentation for human maintainers
- Describing infrastructure, deployment, or runtime configuration (Specs cover source code logic)

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| Spec = requirements document | A Spec describes what code currently does; a requirements doc describes what it should do |
| Spec = specification (formal document) | CoreStory uses "Spec" specifically to mean a machine-generated code behavior description |
| Specs must be reviewed and approved by humans | Specs are derived automatically; they may be reviewed but are not authored through a human approval process |
| One Spec covers an entire application | Specs are atomic — one per component, function, or feature boundary |

## Example

CoreStory generates the following Spec for a `calculateShippingCost` function:

```
Purpose: Calculates the shipping cost for an order based on weight, destination zone, and active promotions.
Inputs: orderWeight (float, kg), destinationZone (enum: DOMESTIC | INTERNATIONAL), promoCode (string | null)
Outputs: shippingCost (float, USD)
Business rules: Free shipping if orderWeight < 0.5kg and DOMESTIC. International orders apply a 1.4x multiplier. PromoCode "FREESHIP" overrides to $0.00.
Dependencies: PromotionService.validateCode(), ZoneRateTable
```

An AI agent receiving this Spec before editing the function has accurate behavioral context without reading the source.

## Related Pages

- [→ Spec-Driven Development (SDD)](../sdd)
- [→ Code Intelligence](../code-intelligence)
- [→ Intelligence Model](../intelligence-model)

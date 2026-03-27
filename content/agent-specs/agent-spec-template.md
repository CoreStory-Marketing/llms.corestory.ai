---
title: CoreStory Agent Spec Format
slug: agent-spec-template
type: spec-template
owner: Ray Ploski
last_reviewed: 2026-03-26
---

# CoreStory Agent Spec Format

## Purpose

The CoreStory Agent Spec format defines a recommended structure for representing code component specifications in a way that AI coding agents can consume effectively. A Spec produced in this format provides a behavioral contract of a code component: what it does, what it depends on, what rules it enforces, and what constitutes correct behavior. Agents consuming a Spec in this format can generate, modify, or review code with accurate grounding.

This format is a recommended convention for teams that want to maintain structured, per-component specs alongside their source code. CoreStory's primary outputs — PRDs and Technical Specifications — provide project-level intelligence that can be retrieved via the MCP endpoint using `get_project_prd` and `get_project_techspec` with section filtering. The Agent Spec format below is a complementary, component-level format that teams can adopt for fine-grained agent grounding.

## Schema

```yaml
module: string                    # The module or package containing this component
component_name: string            # The name of the component as it appears in source
component_type: function | class | service | route  # The structural type of the component
description: string               # Plain-language description of what the component does

file_path: string                 # Repository-relative path to the source file

dependencies:
  - string                        # List of internal or external dependencies (module or function names)

inputs:
  - name: string                  # Parameter or argument name
    type: string                  # Data type (language-native or descriptive)
    description: string           # What this input represents and any constraints

outputs:
  - name: string                  # Return value or output field name
    type: string                  # Data type
    description: string           # What this output represents

preconditions:
  - string                        # Conditions that must be true before this component is called

postconditions:
  - string                        # Conditions that will be true after this component returns successfully

error_cases:
  - condition: string             # The scenario that triggers an error
    behavior: string              # What the component does in response (throw, return, log, etc.)

business_rules:
  - string                        # Domain rules and constraints enforced by this component

examples:
  - scenario: string              # Description of the example scenario
    input: string                 # Representative input values (inline or YAML map)
    expected_output: string       # Expected output for the given input

last_reviewed: YYYY-MM-DD        # ISO date of last Spec review or re-generation
```

## Filled Example

The following is an example Spec for a `calculateShippingCost` function in a retail order management system:

```yaml
module: shipping
component_name: calculateShippingCost
component_type: function
description: >
  Calculates the total shipping cost for an order based on weight, destination zone,
  and any active promotional codes. Applies zone-specific rate multipliers and
  validates promotional codes against the PromotionService before applying discounts.

file_path: src/shipping/cost_calculator.py

dependencies:
  - PromotionService.validateCode
  - ZoneRateTable
  - ShippingAuditLogger

inputs:
  - name: order_weight
    type: float
    description: Total weight of the order in kilograms. Must be greater than 0.
  - name: destination_zone
    type: enum[DOMESTIC, INTERNATIONAL]
    description: Shipping destination zone. Determines base rate and multiplier.
  - name: promo_code
    type: string | null
    description: Optional promotional code. Passed to PromotionService.validateCode for validation. Null if no code is provided.

outputs:
  - name: shipping_cost
    type: float
    description: Total shipping cost in USD, rounded to two decimal places. Returns 0.00 if the FREESHIP promo is valid.

preconditions:
  - order_weight must be greater than 0
  - destination_zone must be a valid enum value (DOMESTIC or INTERNATIONAL)
  - ZoneRateTable must be initialized and populated

postconditions:
  - Returns a non-negative float representing the shipping cost in USD
  - All calculations are logged to ShippingAuditLogger regardless of outcome
  - PromotionService.validateCode is called if and only if promo_code is not null

error_cases:
  - condition: order_weight is 0 or negative
    behavior: Raises ValueError with message "order_weight must be greater than 0"
  - condition: PromotionService.validateCode returns an error (service unavailable)
    behavior: Logs the error to ShippingAuditLogger and proceeds without applying the discount; does not raise
  - condition: destination_zone is not a valid enum value
    behavior: Raises TypeError with message "destination_zone must be DOMESTIC or INTERNATIONAL"

business_rules:
  - Orders with order_weight less than 0.5 kg and destination_zone DOMESTIC receive free shipping (returns 0.00)
  - INTERNATIONAL orders apply a 1.4x multiplier to the base DOMESTIC rate from ZoneRateTable
  - Promotional code "FREESHIP" overrides all calculated costs and returns 0.00, regardless of zone or weight
  - Promotional discounts are applied after zone multipliers
  - All shipping cost calculations are logged to ShippingAuditLogger, including the inputs and the final cost

examples:
  - scenario: Lightweight domestic order with no promo code
    input: "order_weight=0.3, destination_zone=DOMESTIC, promo_code=null"
    expected_output: "shipping_cost=0.00 (free shipping rule applies: weight < 0.5kg, DOMESTIC)"
  - scenario: Standard international order
    input: "order_weight=2.5, destination_zone=INTERNATIONAL, promo_code=null"
    expected_output: "shipping_cost=ZoneRateTable.base_rate(2.5) * 1.4, rounded to 2 decimal places"
  - scenario: Domestic order with FREESHIP promo code
    input: "order_weight=5.0, destination_zone=DOMESTIC, promo_code='FREESHIP'"
    expected_output: "shipping_cost=0.00 (FREESHIP override applied)"

last_reviewed: 2026-03-26
```

## How to Populate

This format can be populated from CoreStory's project-level outputs:

**Via MCP — retrieve project specifications:**

```json
{
  "tool": "corestory__get_project_techspec",
  "arguments": { "project_id": 371, "sections": ["api_specifications"] }
}
```

The Technical Specification includes architectural details, API contracts, data models, and behavioral descriptions that can be distilled into per-component specs in this format.

**Via CoreStory conversations:**

```json
{
  "tool": "corestory__send_message",
  "arguments": { "conversation_id": "abc123", "message": "Describe the calculateShippingCost function including its inputs, outputs, dependencies, and business rules." }
}
```

CoreStory's AI provides answers grounded in the actual codebase, which can be structured into this format.

**Via CoreStory dashboard:**

Navigate to the project in the CoreStory dashboard and review the Technical Specification and PRD. Extract component-level details into this YAML format for inclusion in agent prompts or repository storage.

## Where to Store

Store Specs in a `/specs/` directory at the repository root, one file per component. Use the component's kebab-case name as the filename.

```
repository-root/
  specs/
    calculate-shipping-cost.yaml
    order-fulfillment-service.yaml
    payment-refund-service.yaml
    inventory-reservation-service.yaml
```

Storing Specs in version control alongside the source code ensures:
- Agents can load Specs directly from the repository without a live MCP call
- Spec history is tracked alongside code history in version control
- CI/CD pipelines can validate that Specs are updated when the components they describe are modified

After re-ingestion, review and update the existing Spec files based on the refreshed CoreStory specifications.

## Integration with Playbooks and Agent Setup

Each agent-specific integration guide describes how to connect to CoreStory's MCP server and retrieve specifications:

- **Claude Code**: Use `get_project_techspec` via MCP to retrieve specification sections before coding tasks. See [CoreStory + Claude Code Integration Guide](../playbooks/playbook-claude).
- **Cursor**: Configure MCP in `.cursor/mcp.json` and use `@corestory` to retrieve specifications. See [CoreStory + Cursor Integration Guide](../playbooks/playbook-cursor).
- **Devin**: Upload context to Devin's knowledge base and configure MCP for on-demand retrieval. See [CoreStory + Devin Integration Guide](../playbooks/playbook-devin).

## Related Pages

- [→ Spec](../definitions/spec)
- [→ Intelligence Model](../definitions/intelligence-model)
- [→ Agent Grounding](../definitions/agent-grounding)
- [→ MCP (Model Context Protocol)](../definitions/mcp)
- [→ RAG Context](../definitions/rag-context)

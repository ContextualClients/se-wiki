# Single Start Node

#decisions #architecture #andrew #flow-structure

Use one contextual-start node per flow. Filter inside the flow with switch nodes.

## Decision

One contextual-start node (with empty or minimal condition) per flow. Do not use multiple start nodes to split event types.

Source: AB, architectural review 2026-04-06

## Rationale

Two unfiltered start nodes = unnecessary complexity. The routing logic belongs inside the flow, not at the entry point.

The pattern: catch-all start -> switch node -> branch per event type. This is more readable, testable, and debuggable.

## Context

Previous NetCHB xml-builder flow had 2 contextual-start nodes (one filtered, one catch-all). AB flagged this as over-engineering during the 2026-04-06 review.

## Also: Condition Syntax is Fragile

JSONata expressions on contextual-start nodes work on alba-dev but fail on new tenants (confirmed by AB 2026-04-03).

Leaving the condition empty and filtering inside the flow is more reliable and easier to debug.

Source: expertise.yaml contextual_start_conditions

## Exception

The flow-topic platform requirement is TWO contextual-start nodes for the platform to function (one filtered, one catch-all). This is a platform constraint, not a design choice. The "single start node" direction refers to not adding a third or using multiple filtered starts when one would do.

## Related

- [[platform/flow-structure]] -- entryPoint and start node structure
- [[platform/triggers]] -- contextual-start condition fragility
- [[people/ab]] -- decision source

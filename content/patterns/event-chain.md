# Event Chain

#patterns #events #flow-topic #architecture

The standard pattern for chaining records via post-insert triggers. This is how the NetCHB connector passes data between agents without direct coupling.

## Pattern

```
ECD record created
  -> postInsert trigger fires
  -> netchb-xml-builder reads event, transforms to XML
  -> creates outbound-xml-document record
    -> postInsert trigger fires
    -> netchb-connector reads event, wraps SOAP, sends to NetCHB
    -> stores result in netchb-send-results record
      -> postInsert trigger fires
      -> filing-state-updater reads event, updates ECD state
```

Each agent is independent. They communicate via records, not direct calls.

Source: expertise.yaml event_chain, architecture.solution

## Record as Contract

The outbound-xml-document record is the contract between xml-builder and connector. It carries:
- Processed XML
- Target endpoint
- Operation type
- Client/config reference

This decoupling means either agent can be updated independently.

## msg.event vs msg.payload

Trigger data may arrive in either location. Always use:
```js
var data = msg.event || msg.payload;
```

Source: expertise.yaml msg_event_vs_payload

## Service Version Tracking Gotcha

When deleting/recreating agents, the service loses the version reference. Must:
1. Remove agent from service
2. Re-add it
3. Release

Version mismatch causes agent to run stale cached flow code even after flow PUT updates.

Source: expertise.yaml service_version_tracking

## Agent Separation Rationale

Three separate agents (xml-builder, connector, state-updater) instead of one large agent because:
- Each has distinct responsibilities and secrets
- Failures are isolated and retriable per stage
- xml-builder has no secrets; connector has SOAP credentials (see [[agent-secrets]])
- Independent versioning and deployment

## Related

- [[triggers]] -- postInsert trigger format and creation order
- [[flow-structure]] -- flow-topic agent structure requirements
- [[cron-polling]] -- inbound polling as complement to outbound event chain
- [[http-api]] -- query-api as sync alternative for on-demand reads

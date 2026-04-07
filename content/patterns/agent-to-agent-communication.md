# Agent-to-Agent Communication

#patterns #platform #flow-design #architecture

Don't use HTTP calls between agents. Use the outbox / work request pattern.

## The Problem with HTTP Between Agents

- **Timeout risk** -- slow secondary flow causes HTTP timeout
- **Security exposure** -- HTTP endpoint is publicly accessible
- **Network dependency** -- network hop can fail independently

Raised by ML in [#solution-engineering](https://contextualgroup.slack.com/archives/C09ESB79B0X/p1774465610792659) (2026-03-25).

## Recommended: Outbox / Work Request Pattern

Write a record, let the second agent trigger from it:

```
Agent 1: [finish step 1] -> [create work-request: status=PENDING] -> [End]
Agent 2: [triggered by work-request PENDING] -> [process] -> [update status=COMPLETE]
```

**Benefits:**
- No timeout -- record persists regardless of processing time
- No public endpoint -- Agent 2 triggered by platform
- No network hop -- communication through Native Object store
- Built-in retry -- failed work stays PENDING
- Auditable -- full status history

This is exactly what our NetCHB connector does: xml-builder creates outbound-xml-document (the "work request"), connector triggers from it.

## When You Need Synchronous Response

Poll the record:
```
[create work-request] -> [Delay: 5s] -> [get record] -> [Switch: COMPLETE?]
  -> no: loop back (max N retries)
  -> yes: continue with result
```

Max 12 x 5s = 60 seconds, then error.

## Send to Agent Node

Platform's **Send to Agent** node is fire-and-forget. Use for notifications and side-effects that don't block the main flow.

Source: [SE repo - agent-to-agent-communication.md](https://github.com/ContextualClients/solution-engineering/blob/add/repo-structure-and-phase-0/docs/patterns/flow-design/agent-to-agent-communication.md)

## Related

- [[event-chain]] -- our implementation of the outbox pattern
- [[config-driven-routing]] -- outbound doc carries routing config as the contract
- [[idempotency-guard]] -- work request needs status gate too
- [[alba-netchb]] -- live example: outbound-xml-document is the work request

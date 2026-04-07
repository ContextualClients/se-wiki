# Idempotency Guard

#patterns #platform #original-8 #flow-design

Any flow triggered by a Native Object event must handle duplicate events. One of the "Original 8" non-negotiables.

## The Status Gate

Check status at the very start, before doing anything:

```
[Trigger] -> [get record] -> [Switch: check status]
  -> COMPLETE or PROCESSING? -> End (already handled)
  -> Otherwise -> continue
```

## Set PROCESSING Before Work

Immediately after passing the gate, set PROCESSING before starting actual work:

```
[Switch: not processed] -> [Change: status=PROCESSING] -> [update record] -> [do work...]
```

This prevents a second concurrent trigger from processing the same record.

## Test It

Before production, submit the same record twice within seconds. Confirm:
- Only one output record created
- Second trigger exits via status gate
- No duplicate writes to external systems

Source: [SE repo - idempotency-guard.md](https://github.com/ContextualClients/solution-engineering/blob/add/repo-structure-and-phase-0/docs/patterns/flow-design/idempotency-guard.md) | Part of the "Original 8" in [SE Design & Build Guidelines](https://docs.google.com/document/d/1Am1kmKUjONIZfWh2DgYoSVp18tnT989VEnKzUv3oAdg)

## Related

- [[error-handling]] -- error path after failed processing
- [[correlation-id]] -- track across retry attempts
- [[pre-release-checklist]] -- idempotency test is a release gate
- [[alba-netchb]] -- connector currently lacks this (open item)

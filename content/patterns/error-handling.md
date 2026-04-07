# Error Handling

#patterns #error #catch #andrew

AB's confirmed pattern: catch nodes to end, not contextual-error.

## The Pattern

Two catch nodes, both wired to `contextual-end`:

```
[catch node: scope=all]    -> log-tap (level="error") -> contextual-end
[catch node: scope=uncaught errors] -> log-tap (level="error") -> contextual-end
```

`contextual-error` is NOT in common use. Do not use it.

Source: AB 2026-04-06

## Reference

cargowise-connector on alba-dev: 0 `contextual-error` nodes. Catch (all) + catch (errors) -> contextual-end. This is the production pattern.

Platform training docs say to use `contextual-error` -- that's wrong in practice.

Source: expertise.yaml error_handling_pattern, andrew_flow_standards_20260406

## Log-Tap on Error Paths

Error path log-tap must use `level="error"` for monitoring visibility. Without it, errors are invisible in monitoring.

See [[log-tap-behavior]] for the full level-visibility rules.

## Lint Rule

Every path from `contextual-start` must reach `contextual-end` or `contextual-error`. The linter enforces this. Catch nodes wired to `contextual-end` satisfy this requirement.

## patch op: "set" vs "replace"

Platform training uses `op: "set"` for PATCH operations. Our code uses RFC 6902 `op: "replace"`. Both appear to work. Use "replace" (RFC 6902 standard) unless issues arise.

Source: expertise.yaml patch_op_set_vs_replace

## SE Framework Standard (Original 8)

From the SE Design & Build Guidelines -- error handling is one of the "Original 8" non-negotiables.

**Full error chain:**
```
[Catch: scope=all] -> [Log Tap: ERROR + correlationId] -> [Change: status=ERROR] -> [Native Object: update record] -> [End]
```

The key addition vs AB's simple pattern: **write the error status back to the record** so it's visible in the platform and doesn't silently disappear.

**Error types:**

| Type | Retry? | Route to |
|------|--------|----------|
| Transient (timeout, 429) | Yes, exponential backoff, max 3 | Retry queue, then exception bucket |
| Unrecoverable (bad input, schema mismatch) | No | Exception queue directly |
| Low-confidence AI result | No | Human review queue |

**Exception queue rules:**
- Named human owner (a real person, not a team)
- Defined SLA for review
- Clear action: correct + resubmit, escalate, or reject

**Don't:**
- Rely on global flow error handler only -- it swallows context
- Let errors silently set a stuck state with no status update
- Retry unrecoverable errors

Source: [SE repo - error-handling.md](https://github.com/ContextualClients/solution-engineering/blob/add/repo-structure-and-phase-0/docs/patterns/flow-design/error-handling.md) | Part of the "Original 8" in [SE Design & Build Guidelines](https://docs.google.com/document/d/1Am1kmKUjONIZfWh2DgYoSVp18tnT989VEnKzUv3oAdg)

## Related

- [[log-tap-behavior]] -- error level visibility
- [[flow-structure]] -- lint rules, every path must complete
- [[decisions/no-contextual-error]] -- architectural decision record
- [[idempotency-guard]] -- status gate prevents duplicate processing
- [[correlation-id]] -- include in every error log

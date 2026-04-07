# No contextual-error

#decisions #architecture #andrew #error-handling

Do not use the `contextual-error` node. Wire catch nodes directly to `contextual-end`.

## Decision

Error paths: catch (all) + catch (uncaught errors) -> log-tap (level="error") -> contextual-end.

The `contextual-error` node is not in common use on the platform.

Source: AB, architectural review 2026-04-06

## Rationale

AB's exact words: "contextual-error not in common use. Catch can go to End."

The cargowise-connector on alba-dev (the production reference) has 0 `contextual-error` nodes. This is the pattern in production.

## Context

Platform training documentation mentions `contextual-error` and implies it should be used for error paths. In practice, the platform team does not use it. AB's direction overrides the training docs.

Source: expertise.yaml error_handling_pattern, andrew_flow_standards_20260406

## Implementation

```
[catch scope=all]           -> [log-tap level="error"] -> [contextual-end]
[catch scope=uncaught]      -> [log-tap level="error"] -> [contextual-end]
```

Log-tap density on error path is dealer's choice. Use `level="error"` so errors appear in monitoring.

## Related

- [[patterns/error-handling]] -- full implementation pattern
- [[platform/log-tap-behavior]] -- why level="error" matters
- [[platform/flow-structure]] -- lint rule: all paths must reach end
- [[people/ab]] -- decision source

# Error Handling

#patterns #error #catch #andrew

Andrew's confirmed pattern: catch nodes to end, not contextual-error.

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

## Related

- [[log-tap-behavior]] -- error level visibility
- [[flow-structure]] -- lint rules, every path must complete
- [[decisions/no-contextual-error]] -- architectural decision record

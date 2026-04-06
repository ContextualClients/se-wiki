# Log Tap Behavior

#platform #logging #monitoring #visibility

Log-tap node visibility is NOT what you expect. The destination controls who sees what, and level determines monitoring visibility independently.

## toConsole vs toSideBar

| Setting | Where it goes | Who sees it |
|---|---|---|
| `toConsole` | Backend system | Nasser / platform team only. NOT agent logs. |
| `toSideBar` | Flow editor debug pane | Only visible in the flow editor while open. |

Neither goes to monitoring automatically. **Monitoring visibility is purely level-based.**

Source: Andrew Brooks, 2026-04-06

## Level-Based Monitoring Rules

- `warn` + `error` -- always visible in monitoring logs
- `info` + `debug` -- only visible if agent log level is temporarily elevated in UI (no API for this)

**Rule of thumb:** Use `level="warn"` for checkpoints you want to see in monitoring. Use `level="error"` for all error paths.

## node.warn() Does Not Work

`node.warn()` in function nodes does NOT produce visible logs. Use log-tap nodes instead.

Source: Confirmed in alba-netchb, 2026-04-03. Andrew confirmed 2026-04-06.

## Density

Log-tap density is dealer's choice -- however much you need to understand what's happening. Andrew says there is no wrong answer on quantity.

## Deploy Debug Pattern

Check monitoring logs BEFORE and AFTER every deploy. Never assume a deploy worked. If no new logs appear after a test, the agent didn't reload.

## Related

- [[flow-structure]] -- where log-tap nodes fit in flow architecture
- [[agent-secrets]] -- why agent log level must be set manually after recreate
- [[service-dependencies]] -- Update All is required for reload, not just stop/start

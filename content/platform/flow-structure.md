# Flow Structure

#platform #flows #lint #architecture #standards

Rules and standards for Contextual platform flows. Based on AB architectural review and confirmed platform lint requirements.

## Node Types (Use Correct Names)

- Start: `contextual-start` (NOT event-start)
- End: `contextual-end` (NOT event-end)
- Cron agents also use `contextual-start`/`contextual-end` despite docs suggesting otherwise

Source: Confirmed in alba-netchb, expertise.yaml

## Flow-Topic Required Structure

Every flow-topic flow must have:

1. `native-object-config` node -- required for event delivery
2. Two `contextual-start` nodes -- one filtered, one catch-all with empty condition
3. `catch` node(s) -- wired to `contextual-end` (not contextual-error)
4. `entryPoint` set on agent creation to the contextual-start node ID

See [[triggers]] for why empty condition on catch-all is preferred.

## Single Start Node Direction (Andrew)

AB 2026-04-06: "2 unfiltered start nodes = unnecessary complexity." Use 1 start node unless you have a specific reason for 2. Filter inside the flow with switch nodes.

See [[decisions/single-start-node]] for the full rationale.

## entryPoint CRITICAL

Flow-topic agents created via API MUST include `entryPoint` field set to the contextual-start node ID. The UI sets this automatically. The API does not.

Without it: FATAL "entryPoint '' not found" crash on every event.

Source: ECD blocker resolution, 2026-04-01

## Lint Rules

Platform ESLint runs on all function nodes. Common failures:

- Every http-in must reach http-response in ALL code paths
- Function nodes with 2 outputs: BOTH must trace to completion
- No unused variables (prefix with `_` if intentionally unused)
- No `process.env` -- use `env.get()`
- No `process.uptime` or other process globals
- Every path from contextual-start must reach contextual-end or contextual-error

Source: feedback_deployment_process.md, confirmed multiple deploys

## Logging Standards (Andrew)

- Never log the complete message object permanently at any level
- OK short-term for debugging, remove after
- Log specific properties only (milrose pattern)
- See [[log-tap-behavior]] for level rules

## responseProperty

- `get-native-object` -- use a name like "response"
- `patch-native-object` -- use a name like "response"
- `search-native-object` -- do NOT include responseProperty at all
- Empty string `""` causes "Invalid property expression: zero-length" error

## Deploy Cycle

1. PUT flow (new version via management API, strip `_metaData` first)
2. PUT agent with updated flow#version reference (skip if secrets present)
3. PUT `/p/services/services-dependencies/{tenant}/all`
4. Check monitoring logs

## Related

- [[log-tap-behavior]] -- logging standards
- [[error-handling]] -- catch node pattern
- [[native-object-nodes]] -- native-object-config placement
- [[service-dependencies]] -- deploy chain detail
- [[triggers]] -- trigger/entryPoint interaction
- [[decisions/single-start-node]]

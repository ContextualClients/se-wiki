# Native Object Nodes

#platform #native-object #nodes #bugs

Platform nodes for interacting with native object records. Several have non-obvious behaviors and hard-to-debug failures.

## Node Types

| Node | Purpose | Notes |
|---|---|---|
| `create-native-object` | Create a record | Silently fails if payload has extra fields |
| `get-native-object` | Fetch by ID | Use `objectIdType=str` for known string IDs |
| `search-native-object` | Query with filters | Does NOT support `responseProperty` |
| `patch-native-object` | Update fields (RFC 6902) | Needs full config including `ifHashMatch` |
| `query-native-object` | Does NOT exist | Use search-native-object or get-native-object |

Source: expertise.yaml, confirmed multiple dates

## Reserved msg Property Names (CRITICAL BUG)

`patch-native-object` uses `msg.objectId` as its default internal property. `query-native-object` uses `msg.query`.

If you set these in a function/change node AND configure the same name in the node config, the value resolves to `undefined`.

**Fix: never use these as your property names.**

```
BAD:  msg.objectId = "abc123"  ->  node config: objectId="objectId"
GOOD: msg.targetId = "abc123"  ->  node config: objectId="targetId"

BAD:  msg.query = {...}  ->  node config: query="query"
GOOD: msg.queryString = {...}  ->  node config: query="queryString"
```

Source: Jeff Hagins confirmed 2026-04-03. Nasser confirmed. Jira bug filed.

Workaround: `msg.objectIdType = "str"` forces literal interpretation, or just use a different property name.

## responseProperty Rules

- `patch-native-object` -- supports `responseProperty`
- `get-native-object` -- supports `responseProperty`
- `search-native-object` -- does NOT support `responseProperty`. Including it causes "notification.warnings.undefined" in flow editor.

Source: Confirmed 2026-04-06, query-api v5 fix.

## patch-native-object Required Fields

Node requires all of: `nativeObjectConfig`, `typeId`, `typeIdType`, `objectId`, `objectIdType`, `property`, `propertyType`, `outputProperty`, `ifHashMatch` (empty string ok), `responseProperty`, `parallelism` ("none" not "1"), `maxParallelism`.

Missing `ifHashMatch` or wrong parallelism value causes agent crash backoff loop.

Payload must be RFC 6902 array: `[{op: "replace", path: "/field", value: {...}}]`

Source: SolutionAI snippet + v11 crash fix, expertise.yaml

## create-native-object Silent Failure

If `msg.payload` contains properties not in the object type schema, the create node silently fails -- no error, no record, flow continues.

**Fix:** ensure every field in payload exists in schema, or use `additionalProperties: true`.

Source: expertise.yaml, platform_training_patterns

## native-object-config Node (flow-topic flows)

Flow-topic flows MUST include a `native-object-config` node:
```json
{
  "id": "default-native-object-config",
  "type": "native-object-config",
  "name": "<tenant>"
}
```

Source: [[flow-structure]], entrypoint_required pattern

## Related

- [[flow-structure]] -- native-object-config placement in flow
- [[schema-rules]] -- schema must match exactly or create fails silently
- [[native-object-nodes]] -- patch RFC 6902 format

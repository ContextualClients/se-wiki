# Alba-Dev as Reference Tenant

#decisions #architecture #andrew #schemas

alba-dev is the source of truth for object type schemas and the production event-driven flow reference.

## Decision

Always pull object type schemas from alba-dev. Never create them from scratch or simplify them. The transform adapts to the schema, not the other way around.

Source: AB 2026-04-03, JH confirmed

## Rationale

The NetCHB connector is packaged as a service with object types as peer dependencies. If the types don't match the destination tenant, service installation fails. Schemas must be identical to the source.

Additionally, alba-dev is the reference for production event-driven patterns. Milrose is for deep research and report formatting -- different use case.

## What Happened Without This Rule

- Created ECD with 38 fields (correct: 52)
- Created ISF with 20 fields (correct: 82)
- Both simplified to fit the transform function
- Result: service installation would fail, wrong records created

Source: feedback_never_create_schemas_from_scratch.md

## How to Apply

```
GET /p/native-object-registry/types/{type-id}
```

Run this against alba-dev. Use result as-is. Strip only `_metaData`, `triggers`, `actions`, and `relationships` to non-existent types.

## What Was Replaced

2026-04-03: ECD (52 fields), ISF (82 fields), xml-template (3 fields) all replaced with exact alba-dev copies.

ISF required stripping `schema.relations` (cargowise-company, cargowise-branch, organization references that don't exist on the new tenant).

## Alba-Dev vs Milrose

| Tenant | Type | Use as reference for |
|---|---|---|
| alba-dev | Production event-driven | Event-driven patterns, schemas, connector architecture |
| milrose | Deep research + report formatting | NOT high-volume event-driven patterns |

Source: AB 2026-04-06

## Related

- [[platform/schema-rules]] -- implementation guide
- [[clients/alba-netchb]] -- current schema state
- [[people/ab]] -- decision source
- [[people/jh]] -- confirmed the peer dependency model

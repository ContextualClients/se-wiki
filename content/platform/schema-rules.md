# Schema Rules

#platform #schema #object-types #andrew

Never create object type schemas from scratch. Always pull from the source tenant.

## The Rule

Pull exact schema from alba-dev (or the reference tenant). Use it as-is.

Source: AB 2026-04-03, JH confirmed

## Why It Matters

The NetCHB connector is packaged as a service with object types as peer dependencies. If the object types on the destination tenant don't match the service's peer dependencies, installation fails.

The transform adapts to the schema. The schema never adapts to the transform.

## What Happened Without This Rule

Created ECD with 38 fields (should be 52). Created ISF with 20 fields (should be 82). Simplified to fit the transform. That's backwards -- broke service installation.

Source: feedback_never_create_schemas_from_scratch.md

## How to Pull Schema

```
GET /p/native-object-registry/types/{type-id}
```

Use management proxy. NOT the direct registry URL.

## What to Strip

Strip only:
- `_metaData`
- `triggers`
- `actions`  
- `relationships` to types that don't exist on target tenant

Keep everything else: all properties, state enums, required fields, definitions, features.

## What to Keep

- All properties (including ones you don't use)
- State enums (must match source exactly)
- Required fields
- Definitions / $defs
- Features: `{"auditTrail": {"enabled": true}, "version": {"enabled": false}}`

## Object Type Internal Requirement

`objectType` must be `"internal"` not `"external"` for record CRUD. External types return "No rule for [create] found".

Source: ECD blocker 2026-04-01, expertise.yaml object_type_internal

## Schema Replaced in alba-netchb

2026-04-03: ECD (52 fields), ISF (82 fields), xml-template (3 fields) all replaced with exact alba-dev copies. ISF required stripping `schema.relations` (cargowise-company, cargowise-branch, organization references).

## Related

- [[clients/alba-netchb]] -- current schema state
- [[decisions/alba-dev-as-reference]] -- why alba-dev is source of truth
- [[native-object-nodes]] -- schema mismatch causes silent create failure

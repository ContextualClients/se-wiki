# Service Dependencies

#platform #service #deployment #api

The service is the glue between components. Getting it wrong means agents run stale code or don't receive events.

## Why It Matters

- A service must exist on tenant for triggers to deliver events
- New agents/flows/object types added to service are NOT reloaded by stop/start
- Version mismatch in service causes agent to run stale cached flow code

## Service Create

```
POST /p/services/services
{"id": "<tenant>", "name": "<tenant>", "dependencies": {}}
```

## Add Dependency via API (No UI Required)

Captured from UI network trace 2026-04-05. Two-step process:

**Step 1: Ownership check**
```
POST /p/services/services-dependencies/{tenant}/are-owned-by-other-service
[{"typeId": "agent", "version": 1, "instanceId": "agent-name"}]
```

**Step 2: Add dependency**
```
PATCH /p/services/services-dependencies/{tenant}
{
  "serviceId": "{tenant}",
  "dependencies": {
    "direct": [
      {"typeId": "agent", "instanceId": "agent-name", "version": 1, "op": "add"}
    ],
    "peer": []
  }
}
```

For object types: `typeId` IS the object type ID (e.g. "extended-customs-declaration"), no `instanceId` field. For agents/flows: `typeId="agent"` or `"flow"`, `instanceId=name`.

Source: Captured from UI network trace, confirmed 2026-04-05

## Force Reload (Update All)

Stop/start does NOT reload flow code. Use:

```
PUT /p/services/services-dependencies/{tenant}/all
```

Headers: Authorization + x-org-id. No body needed.

Source: Discovered from UI network trace 2026-04-03

## Deploy Chain

1. PUT flow (new version)
2. PUT agent with new flow#version (skip if agent has secrets -- use UI)
3. PUT `/p/services/services-dependencies/{tenant}/all`
4. Test

## Caveat on New Components

"Update All" only updates EXISTING components. New components (never added before) still require UI to add to service. Use the Add Dependency API above as the alternative.

## Related

- [[agent-secrets]] -- why you skip the agent PUT step for agents with secrets
- [[flow-structure]] -- full deploy cycle context
- [[triggers]] -- triggers must exist before service delivers events

# HTTP API Patterns

#patterns #http #flow-http #soap #rest

Patterns for external HTTP calls and internal REST APIs on the platform.

## flow-http Agent

Agent type for exposing REST endpoints. Handles HTTP requests, returns responses.

**URL pattern:**
```
https://{agent-name}.service.{tenant}.my.contextual.io/{path}
```

Note the `.service.` segment. Without it you get nginx 404.

Example: `https://netchb-query-api.service.alba-netchb.my.contextual.io/api/health`

The URL is also shown on the agent definition tab in the UI.

Source: expertise.yaml flow_http_agent_url

## flow-http Size UUID

Use `"97c7d433-7729-4800-ac80-086ade746541"` for flow-http agents.
(flow-topic agents use `"8836d51b-51f0-4417-b799-b8fb692e6a1b"`)

Source: expertise.yaml agent_size_uuid

## Internal REST (netchb-query-api)

5 endpoints exposed by query-api v5:
- `/api/health`
- Records read mode (uses native object get/search)
- Live SOAP mode (queries NetCHB directly)
- Rate limit handling (returns `RATE_LIMITED` status)

Lint rule: every `http-in` must reach `http-response` in ALL code paths.

## SOAP Abstraction

SOAP calls via HTTP node with XML body. Key properties:
- SOAPAction: empty string `""` for ALL NetCHB operations
- All params are `xsd:string`
- Per-call username/password in SOAP body
- Password needs HTML entity encoding (`>` becomes `&gt;`)

Source: expertise.yaml soap_auth

## Management API Auth

- Registry/Tenant API: `native-object.{tenant}.my.contextual.io/api/v1` -- client credentials auth
- Management API: `api.prod-001.prod.contextual.io/p/native-object/*` -- browser JWT + X-Org-Id

Source: project_platform_api.md

## Records PATCH Format

Uses RFC 6902 JSON Patch. Content-Type must be `application/json`.

```json
[{"op": "replace", "path": "/field", "value": "new-value"}]
```

Source: project_platform_api.md, expertise.yaml

## PUT is Full Replace

No PATCH on platform components (flows, agents, object types). Always GET first, modify, then PUT the complete object. Strip `_metaData` before PUT.

## Related

- [[cron-polling]] -- SOAP calls for CBP status
- [[handlebars-templates]] -- XML generation before SOAP call
- [[service-dependencies]] -- management API for deploy chain
- [[agent-secrets]] -- auth model for management vs registry API

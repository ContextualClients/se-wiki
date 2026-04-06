# Alba NetCHB

#client #alba #netchb #customs #filing

Tenant overview for the NetCHB customs filing connector built on alba-netchb tenant.

## What It Does

Event-driven NetCHB connector for ECD (formal entry) and ISF filing. Creates records, transforms to XML via Handlebars templates, submits via SOAP to NetCHB, polls CBP for status, writes back to ECD.

Architecture: `ECD/ISF post-insert -> netchb-xml-builder -> outbound-xml-document -> netchb-connector -> netchb-send-results -> filing-state-updater`

## Tenant

- ID: `alba-netchb`
- Phase: 1 (active build)
- SE Lead: Brian Pyatt
- Parent client: alba
- Last scan: 2026-04-06

## Object Types (6)

| Type | Fields | Purpose |
|---|---|---|
| `extended-customs-declaration` | 52 | ECD records -- main entry record |
| `us-importer-security-filing` | 82 | ISF records |
| `outbound-xml-document` | -- | Intermediate: holds rendered XML + routing config |
| `netchb-send-results` | -- | SOAP response storage |
| `xml-template` | 3 | Handlebars template content |
| `netchb-template-config` | -- | Config: (clientId, objectType) -> (templateId, endpoint) |

Schema source: exact copies from alba-dev. See [[decisions/alba-dev-as-reference]].

## Agents (5 active + 1 legacy)

| Agent | Type | Version | Purpose |
|---|---|---|---|
| `netchb-xml-builder` | flow-topic | v30 | ECD/ISF -> XML transform |
| `netchb-connector` | flow-topic | v3 | SOAP send to NetCHB |
| `filing-state-updater` | flow-topic | v1 | State update on ECD after send |
| `netchb-query-api` | flow-http | v5 | REST API: records + live SOAP |
| `netchb-status-poller` | flow-cron | v13 | Hourly CBP status polling |
| `netchb-demo` | flow-http | v64 | Legacy demo UI (stable baseline) |

## Service

`alba-netchb-v2` -- released v2, 6 object types + active agents.

## Filing Results

- Total ECD filings: 50+, all ACCEPTED
- Total ISF filings: 1 ACCEPTED (ref 121087466)
- Scale test: 11 entries from 10,000 lines, all ACCEPTED (2026-04-03)
- Latest entry: ST9-2074864-9 (2026-04-06)
- Latest ISF: ref 121087466 (2026-04-03)

## NetCHB Sandbox

- URL: `https://sandbox.netchb.com`
- User: `alba-sbx`
- Entry endpoint: `.../main/services/entry/EntryUploadService`
- ISF endpoint: `.../main/services/isf/IsfUploadService`
- SOAPAction: `""` (empty for all operations)
- Rate limit: 1 query per entry per hour

Importer tax ID required for entries to show in NetCHB UI. Sandbox EIN: `71-0415188`.

## Three-Part Architecture (all complete)

1. Outbound XML (ECD+ISF -> XML -> SOAP -> NetCHB) -- DONE
2. Internal API layer (query-api v5, 5 REST endpoints) -- DONE
3. Inbound events (status-poller v13, hourly cron, CBP writeback) -- DONE

## Next Steps

- Production credentials from Descartes
- Idempotency guard on connector (dedup if triggered twice on same outbound doc)
- Error retry / dead letter pattern
- Additional entry type templates (In-Bond, FTZ, Quota)
- ISF Handlebars template (currently inline JS -- needs template pattern like ECD)

## Key People

- [[people/jeff-hagins]] -- architecture direction, entry types scope, prod access
- [[people/andrew-brooks]] -- architectural review, flow standards
- [[people/james-stolp]] -- platform issues, cron feedback
- [[people/brian-pyatt]] -- SE lead, builder

## Related

- [[patterns/event-chain]] -- core architecture pattern
- [[patterns/config-driven-routing]] -- template/endpoint routing
- [[patterns/cron-polling]] -- status-poller implementation
- [[decisions/alba-dev-as-reference]] -- schema source of truth

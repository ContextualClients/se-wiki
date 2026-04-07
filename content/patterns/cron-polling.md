# Cron Polling

#patterns #cron #polling #poller #cbp

The inbound half of the NetCHB integration. Polls CBP for status updates and writes back to ECD records.

## Why Polling

NetCHB has NO push webhooks. Must poll `queryEntryStatus`. Rate limit: 1 query per entry per hour on sandbox.

Source: expertise.yaml no_webhooks

## Cron Agent Structure

`netchb-status-poller v13` (flow-cron, hourly schedule).

Uses `contextual-start`/`contextual-end` nodes (not event-start/end, despite some docs suggesting otherwise).

Source: expertise.yaml, confirmed in working implementation

## Fan-Out Loop Pattern

1. Cron fires
2. Query all ECD records in SENT state (need status check)
3. Loop through records (sequential, not parallel -- rate limit)
4. For each: call NetCHB `queryEntryStatus` via SOAP
5. Parse CBP response, patch ECD record with status fields:
   - `cbpState`
   - `status7501`
   - any other CBP-returned fields
6. Loop exits after processing (not infinite -- exits after N entries)

Source: AB feedback 2026-04-06 (poller loop documented in v14 flow info)

## Rate Limiting

- Sandbox: 1 query per entry per hour
- Query API handles this: returns `RATE_LIMITED` status when exceeded
- Poller respects this by running hourly and skipping already-checked entries

## CBP Writeback

Patches ECD `addInfo` field with CBP response data via `patch-native-object`. Uses `msg.targetId` (not `msg.objectId` -- see [[native-object-nodes]] reserved names).

Source: expertise.yaml, poller v13 implementation

## ACCEPTED vs Warnings

ACCEPTED only means transmission received by NetCHB. Warnings in the response are real problems for production -- must be surfaced and tracked.

Source: [[clients/alba-netchb]], expertise.yaml netchb_accepted

## Status Poller vs Query API

- `netchb-status-poller` -- automated cron, writes back to ECD
- `netchb-query-api` -- on-demand REST API for human/system queries

Both use the same underlying SOAP call to NetCHB.

## Related

- [[http-api]] -- SOAP call implementation in query-api
- [[native-object-nodes]] -- patch-native-object for CBP writeback
- [[clients/alba-netchb]] -- poller version history and filing results

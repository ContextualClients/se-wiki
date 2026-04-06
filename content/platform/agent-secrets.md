# Agent Secrets

#platform #agents #secrets #envvars #gotcha

Agent PUT via API destroys existing envVar secrets. This is a critical, confirmed platform behavior.

## The Problem

`GET /p/...agent/{id}` returns `<ENCRYPTED>` for secret values. If you PUT that back, the platform either rejects the encrypted string or wipes the secret entirely.

**Result:** Agent loses credentials, stops working, may start failing silently.

Source: Confirmed 2026-04-03 -- netchb-connector PUT wiped NETCHB credentials.

## Rules

- **NEVER PUT on an agent that has envVar secrets**
- To update flow version on an agent with secrets -- use the UI only
- Only PUT on agents with NO secrets (like netchb-xml-builder)
- **Exception:** EnvVars CAN be set on NEW agents via API PUT (no existing secrets to destroy)

## Workarounds

1. **Use UI only** for agents with secrets. Update flow version via agent edit in the UI.
2. **Service dependency add API** -- update service without touching the agent directly. See [[service-dependencies]].
3. For new agents: PUT includes envVars in the initial create. Safe because there are no existing secrets yet.

## Log Level Reminder

Every time an agent is created, recreated, or restarted: **set log level to DEBUG in the UI**. There is no API for this. Monitoring only shows warn+error by default. See [[log-tap-behavior]].

## Related

- [[service-dependencies]] -- how to update agents without full PUT
- [[log-tap-behavior]] -- log level must be set manually in UI
- [[flow-structure]] -- deploy chain after flow update

# Correlation ID

#patterns #platform #debugging #flow-design

Single most important thing for debugging a specific execution in production. Must flow from first trigger to last output.

## Set at Entry

```javascript
msg.correlationId = msg.correlationId || msg.event._id || msg._msgid;
return msg;
```

Using the record's `_id` ties every log line to a specific record.

## Never Replace msg Entirely

```javascript
// BAD -- drops correlationId
msg.payload = { status: "PROCESSING" };

// GOOD -- preserves everything on msg
msg = { ...msg, payload: { status: "PROCESSING" } };
```

## Include in Every Log

Every Log Tap should reference it:
```jsonata
{ "correlationId": msg.correlationId, "stage": "extraction-complete", "recordId": msg.event._id }
```

Filter by `correlationId` in the log viewer to trace a single execution.

## Pass to External Systems

```javascript
msg.headers = { ...msg.headers, "X-Correlation-Id": msg.correlationId };
```

Source: [SE repo - correlation-id.md](https://github.com/ContextualClients/solution-engineering/blob/add/repo-structure-and-phase-0/docs/patterns/flow-design/correlation-id.md)

## Related

- [[error-handling]] -- include correlationId in error logs
- [[event-chain]] -- correlationId flows across agent boundaries
- [[log-tap-behavior]] -- what shows up in monitoring

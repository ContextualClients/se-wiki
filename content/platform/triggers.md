# Triggers

#platform #triggers #events #flow-topic

Triggers wire object type lifecycle events to agents. The format and order of creation matter.

## Trigger Format (PUT on Object Type)

```json
{
  "type": "system",
  "id": "<UUID string>",
  "name": "<trigger name>",
  "action": "send-to-agent",
  "agentId": "<agent-name>"
}
```

**Critical:** `type` must be `"system"` not `"custom"`. Do NOT include a `ruleSet` field -- adding it causes validation error for system triggers.

Source: expertise.yaml trigger_api_format, confirmed platform testing

## Creation Order

Must create in this order to avoid broken states:

1. Object type (internal) 
2. Flow (with native-object-config + catch-all contextual-start)
3. Agent (WITH entryPoint set)
4. Trigger (AFTER agent exists)
5. Service dependency

Trigger created before agent exists may not wire correctly.

## After Tenant Suspension

Triggers may need to be deleted and recreated via object type PUT. Also run Update All Service Components to reconnect agents to their message topics.

Source: expertise.yaml trigger_api_format

## contextual-start Condition Fragility

JSONata expressions with `=` and `and` work on alba-dev but fail on new tenants.

**Confirmed by Andrew 2026-04-03.**

Recommended workaround: leave contextual-start condition empty, filter inside the flow with switch nodes. More reliable, easier to debug.

Source: expertise.yaml contextual_start_conditions

## msg.event vs msg.payload

In flow-topic event flows, trigger data may arrive in `msg.event` OR `msg.payload` depending on how the flow was created/configured.

**Safe pattern:**
```js
var data = msg.event || msg.payload;
```

Source: expertise.yaml msg_event_vs_payload

## Related

- [[flow-structure]] -- entryPoint and start node requirements
- [[event-chain]] -- full event-driven pattern with triggers
- [[service-dependencies]] -- service must exist for triggers to deliver

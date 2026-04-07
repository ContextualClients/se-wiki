# Mock Data Strategy

#patterns #testing #flow-design

How to test flows that call external APIs with no sandbox environment.

## Mock Toggle

Add a `useMock` flag that routes around the real API:

```
[entry] -> [Switch: msg.useMock?]
  -> yes: [Change: inject mock response] -> [continue]
  -> no: [HTTP Request: real API] -> [continue]
```

Store in flow config or env var:
```javascript
msg.useMock = msg.config?.useMock || env.get("USE_MOCK_API") === "true" || false;
```

## Mock Data Storage

| Option | When |
|--------|------|
| Inline in Change node | Simple, stable responses |
| Native Object record | Complex or varying test cases |
| Separate mock flow agent | Multiple flows call same API |

## Dev vs Prod

Mock mode must be **impossible in prod**. Use environment-level flags only available in dev.

Raised by ML in [#solution-engineering](https://contextualgroup.slack.com/archives/C09ESB79B0X/p1774462124872649) (2026-03-25) re: FranchiseFastlane external API with no test environment.

Source: [SE repo - mock-data-strategy.md](https://github.com/ContextualClients/solution-engineering/blob/add/repo-structure-and-phase-0/docs/patterns/testing/mock-data-strategy.md)

## Related

- [[pre-release-checklist]] -- mock toggle must be off before release
- [[error-handling]] -- test error paths with mock error responses

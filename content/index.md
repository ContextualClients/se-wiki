# SE Knowledge Wiki

Drop files into `raw/` and run `/se:wiki-ingest` to process them into linked wiki pages.
Use `/se:wiki-file <topic>` to capture insights from conversations.
Run `/se:wiki-lint` periodically to check health.

---

## Platform
- [[agent-secrets]] -- Agent PUT via API destroys existing envVar secrets. Critical gotcha.
- [[flow-structure]] -- Rules for flows: start nodes, lint, deploy cycle, Andrew's standards
- [[log-tap-behavior]] -- Visibility is level-based, not toConsole/toSideBar. Andrew confirmed.
- [[native-object-nodes]] -- Reserved names, responseProperty, patch config, silent failures
- [[schema-rules]] -- Never create from scratch. Always pull from source tenant (alba-dev).
- [[service-dependencies]] -- Add API (PATCH), Update All endpoint, deploy chain sequence
- [[triggers]] -- Format, system vs custom, ruleSet gotcha, creation order

## Patterns
- [[config-driven-routing]] -- Template configs map filing type to endpoint. Zero flow changes to add types.
- [[cron-polling]] -- Fan-out loop for sequential polling. CBP status writeback to ECD addInfo.
- [[error-handling]] -- Andrew's pattern: catch all + catch uncaught -> contextual-end. No contextual-error.
- [[event-chain]] -- Record-as-contract chaining via postInsert triggers. Agent separation.
- [[handlebars-templates]] -- libs config, 11 custom helpers, template-as-record pattern.
- [[http-api]] -- flow-http URL pattern (.service. segment), SOAP abstraction, RFC 6902.

## Clients
- [[alba-netchb]] -- NetCHB customs connector. 50+ filings, 100% accepted, 3/3 architecture complete.

## Decisions
- [[alba-dev-as-reference]] -- Schema source of truth. Flow patterns reference. Why and how.
- [[karpathy-wiki-comparison]] -- LLM Wiki vs Clarity. Gaps, advantages, bridge plan.
- [[no-contextual-error]] -- Andrew: catch -> end, not contextual-error. Rationale + evidence.
- [[single-start-node]] -- One start per flow. Filter with switch nodes inside. Andrew's direction.

## People
- [[andrew-brooks]] -- Platform architect. Architectural reviews. Flow standards owner.
- [[brian-pyatt]] -- SE lead. Clarity/Forge builder. Open action items tracked.
- [[james-stolp]] -- Platform engineering. Lint rules, cron feedback, internal tooling.
- [[jeff-hagins]] -- Architecture direction. Entry type scope. Schema rules.

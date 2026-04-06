# Andrew Brooks

#people #leadership #ceo #founder

Chief Executive Officer of Contextual Software Inc. 1 year 11 months. Conducts architectural reviews of SE builds and sets standards for how flows should be built.

**Title:** Chief Executive Officer
**Department:** No Department (company-wide)
**Tenure:** 1 year, 11 months
**Direct reports:** 5/28 (PK Krishnamoorthy, Jeff Hagins, Ben Edwards, Justin Corbell + engineering org)

## Role

CEO of Contextual. Founded the company. Hands-on with engineering -- directly reviews SE flow architecture and runs "Sync on Design Approach" sessions. The entire org reports up to him: PK (CCO/Delivery), Jeff (CTO/R&D), Ben (VP People Ops), Justin (Sales), plus the engineering org (Aiden Landers, James Stolp, platform engineers).

## Architectural Review Feedback (2026-04-06)

Full review with Brian on NetCHB connector flows. Key direction:

**Flow structure:**
- 2 unfiltered start nodes = unnecessary complexity. Use 1. See [[decisions/single-start-node]].
- Filter with switch nodes inside the flow, not at the start node condition level.
- Start node filtering: payload-based conditions are default (milrose pattern). Headers-based depends on where critical info is.

**Logging:**
- Never log complete message object permanently at any level. OK short-term for debugging, remove after.
- `node.warn()` is wrong -- use log-tap nodes.
- Error paths need log-tap at `level="error"` for monitoring visibility.
- Log-tap density is dealer's choice.

**Error handling:**
- `contextual-error` not in common use.
- Pattern: catch (all) + catch (uncaught errors) -> contextual-end.
- See [[decisions/no-contextual-error]].

**References:**
- alba-dev is the production event-driven reference (not milrose -- milrose is deep research/report formatting, not high-volume event-driven).
- cargowise-connector on alba-dev = 0 contextual-error nodes.

Source: expertise.yaml andrew_flow_standards_20260406, project_andrew_review_20260406.md

## Confirmed Platform Facts (from Andrew)

- `toConsole` goes to Nasser/platform team backend, NOT agent logs (2026-04-06)
- `toSideBar` is flow editor debug pane only (2026-04-06)
- Monitoring visibility is purely level-based: warn+error always, info+debug only with elevated log level (2026-04-06)
- contextual-start condition syntax is fragile on new tenants (2026-04-03)
- Schema rule: never create from scratch, always pull from alba-dev (2026-04-03)

## Action Items for Brian (from review)

1. Playwright tests for flow editor opening
2. Document poller loop mechanism (why it loops, how it exits)
3. Complete tutorials 1-3 manually + Jeff's weather app
4. Practice manual flow creation in editor

## Related

- [[decisions/single-start-node]]
- [[decisions/no-contextual-error]]
- [[decisions/alba-dev-as-reference]]
- [[platform/log-tap-behavior]]
- [[platform/flow-structure]]
- [[people/jeff-hagins]] -- CTO, peer at leadership level (reports to CEO)
- [[people/aiden-landers]] -- Director of Platform Engineering, reports to Andrew
- [[people/james-stolp]] -- Platform Engineering, reports to Aiden Landers

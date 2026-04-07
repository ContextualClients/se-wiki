# Pre-Release Checklist

#patterns #deployment #original-8 #checklist

Use before every production release. All items must pass. From [SE Design & Build Guidelines S8.3](https://docs.google.com/document/d/1Am1kmKUjONIZfWh2DgYoSVp18tnT989VEnKzUv3oAdg).

## Checklist

- [ ] All test nodes pass in dev
- [ ] Eval dataset run against AI/prompt changes -- results documented
- [ ] All HTTP In endpoints have authentication enabled
- [ ] Dev credentials replaced with prod in all connections
- [ ] No hardcoded secrets in flow nodes or prompts
- [ ] Exception queues configured with named owners + SLAs
- [ ] Audit trail + version control enabled on state-bearing object types
- [ ] Idempotency logic tested with duplicate submissions
- [ ] Prod tenant access list reviewed and approved
- [ ] Debug nodes removed (no `active: true` in flows.json)
- [ ] Rollback plan documented

## Second Pair of Eyes

Every prod release needs a reviewer who didn't build it. They check:
- Does this match the spec/ticket?
- Leftover dev artifacts?
- Security gaps?

**Reviewer sign-off:** Name + timestamp in Jira ticket or release comment.

Source: [SE repo - pre-release-checklist.md](https://github.com/ContextualClients/solution-engineering/blob/add/repo-structure-and-phase-0/docs/guidelines/pre-release-checklist.md)

## Related

- [[idempotency-guard]] -- idempotency test is a release gate
- [[error-handling]] -- exception queues must be configured
- [[correlation-id]] -- must be flowing before release
- [[alba-netchb]] -- current gaps: no auth on query API, no idempotency, no rollback plan

# JS

#people #engineering #platform

Platform engineering team. Joined Andrew's architectural review session. Platform internals knowledge.

**Department:** Engineering
**Note:** Engineering org. Works closely with SE team on tooling questions.

## Role

Internal Contextual platform team under AL (Director of Platform Engineering). Works on platform tooling and flow editor infrastructure. Knowledge of platform lint, validation, and flow binding internals.

## Feedback on NetCHB Review (2026-04-06)

From architectural review with Brian and Andrew:

**Lint and orphaned nodes:**
Previous flow versions had disconnected/orphaned function nodes causing lint errors. Platform should catch these at generation time, not at agent binding. (Implication: lint early in the flow build process, not as an afterthought.)

**SolutionAI vs API approach:**
The platform's native SolutionAI has Zod verification layers that the API endpoint approach doesn't have. Brian's system uses API calls, not the flow editor tunnel or SolutionAI skill.

**Team direction:**
Team is working on a Git-based approach for shared platform knowledge and project-specific context. For V1, flexibility is more important than efficiency -- abstraction can come later.

Source: project_andrew_review_20260406.md

## Platform Items to Ask About

- "sites" capability for UI/RBAC/image handling -- was "next week" as of 2026-04-03. Status unknown.
- Cron agent behavior (agent/flow-cron specific feedback)

Source: expertise.yaml unvalidated_observations

## Related

- [[people/ab]] -- VP of Engineering, co-lead on architectural review
- [[people/jh]] -- CTO
- [[platform/flow-structure]] -- lint and orphaned node issues

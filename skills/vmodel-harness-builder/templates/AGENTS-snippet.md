# AGENTS.md snippet

Add this section to the repository's `AGENTS.md` so agents that do not load skills still follow
the harness. Replace the placeholders.

```
## Delivery process (V-model harness)

For any code change in this repository, follow {{HARNESS_PATH}} from start to finish.
- Anchor the change to a {{TICKET_NOUN}} in {{INTAKE_SYSTEM}} first.
- No product code before the definition package passes independent review and a human approves
  the plan.
- No pull request before every verification gate is green and evidence is attached.
- Never skip silently. Testing, regression and evidence are never skippable.
- Record durable learnings in this file under the relevant area:
  `- <date>: <learning>. Rationale: <why>. Applies to: <scope>.`
```

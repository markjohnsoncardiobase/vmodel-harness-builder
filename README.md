# V-Model Harness Builder

An agent skill that helps you build your own trustworthy delivery harness for coding agents,
using the systems-engineering V-model.

It does not hand you someone else's software factory. It teaches the principles, researches your
codebase, interviews you about your intake, tickets, risks, validation and people, and then
generates a harness that fits your setup.

Companion to the AI Engineer Sydney 2026 talk **"Why Software Factories Can't Be Trusted (and how
the Systems Engineering V-model helps)"** by Mark Johnson.

## Why

A language model is an unreliable component. It is fast and often right, but its output is
stochastic and it will claim success it has not earned. A software factory that pipes tickets into
an agent and pipes PRs out cannot be trusted on that basis alone. Engineers have always built
reliable systems from unreliable parts, using decomposition, independent verification, feedback
loops and failsafes. The V-model packages those ideas: decompose on the left, build at the bottom,
verify each level on the right against its paired level on the left, and put humans where
verifiability is low. Deterministic gates hold the order. Independent adversarial verifiers check
what scripts cannot. Traceability ties every test back to a requirement. This skill helps you
apply that to your own repo.

## What you get

```
skills/vmodel-harness-builder/
  SKILL.md                      entry point and the interactive build flow
  references/
    principles.md               systems-engineering fundamentals for agentic delivery
    v-model.md                  every step of the V, its output and its paired verification
    interview.md                the intake question bank and how answers shape the harness
    gates.md                    deterministic, adversarial and human gates; gate menu by stack
    verifiers.md                adversarial reviewer and completion verifier design and prompts
    rtm.md                      requirements traceability matrix format and rules
  templates/
    harness-SKILL.template.md   the harness skill the builder fills in for you
    AGENTS-snippet.md           for agents that read AGENTS.md instead of skills
    workflow-outline.md         optional orchestration script outline
    artefacts/                  triage, system understanding, ConOps, requirements, test plan,
                                design, specifications, research, plan, RTM, review verdict,
                                plan digest, verification summary, skip disclosure, handback
examples/                       two worked examples of generated harnesses (fictional teams)
```

## Install

Claude Code (personal, all projects):

```bash
git clone https://github.com/markjohnsoncardiobase/vmodel-harness-builder.git
cp -r vmodel-harness-builder/skills/vmodel-harness-builder ~/.claude/skills/
```

Claude Code (one project):

```bash
mkdir -p .claude/skills
cp -r vmodel-harness-builder/skills/vmodel-harness-builder .claude/skills/
```

Other agents (Codex, Cursor, Gemini CLI, and any tool that reads `AGENTS.md`): copy the
`skills/vmodel-harness-builder` folder into your repo (for example `docs/agent-skills/`) and add a
line to your `AGENTS.md`:

```
To build or change our delivery harness, follow docs/agent-skills/vmodel-harness-builder/SKILL.md.
```

The skill uses the Agent Skills format (`SKILL.md` with `name` and `description` frontmatter), so
any tool that supports that format can load it directly.

## Quick start

1. Open your repo in your agent tool.
2. Switch to the strongest reasoning model you have, at the highest thinking effort. Harness
   design is a one-off, high-leverage task. The skill will remind you.
3. Say: **"Build my V-model harness."**
4. The builder will:
   1. recommend the model and effort;
   2. research your codebase (stack, test and lint commands, CI, how to run the app, risky areas);
   3. interview you in small batches (intake, what you build, ticket nature, risk, available
      verification, humans in the loop, budget, triage, artefacts, handback, memory);
   4. show you a one-page design of your V with every gate typed as deterministic, adversarial or
      human, and wait for your approval;
   5. generate your harness skill and artefact templates into your repo;
   6. dry-run it on one small ticket and tune it with you;
   7. remind you of the limits.
5. From then on, use your generated harness for each ticket.

## When to use it

- Production code where a wrong change hurts someone (users, money, data, safety).
- Work arrives as tickets or requests you can anchor to a record.

## When not to use it

- Throwaway prototypes and exploration.
- One-line config changes where the overhead outweighs the risk.
- Non-code tasks.

## Honest limitations

- The V is slower than letting an agent run free. That is the price of evidence.
- Validation at the top of the V ("did we build the right thing?") still needs a human.
- Pieces alone are not the V. Only TDD, or only adversarial review, misses the pairing and
  traceability that make it work.
- The first generated harness will need tuning. Expect to adjust it over your first ten or so
  tickets.

## Reference implementation

The skill is model-agnostic. Claude Code is the reference implementation: skills, subagents with
fresh context for the generator and verifiers, and workflows to hold the stage order in code.
Where a tool lacks one of these, the builder will tell you what the harness does instead.

## Licence

MIT. See [LICENSE](LICENSE).

Author: Mark Johnson

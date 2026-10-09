# Principles: systems engineering for agentic delivery

These principles explain why the harness looks the way it does. Teach them when the user asks
"why do we need this step?". Use them to decide trade-offs during design.

## 1. The model is an unreliable component

A language model produces plausible output. Plausible is not the same as correct. The same prompt
can give a different answer on a second run. The model can also claim work is done, tested or
verified when it is not.

Engineers have faced this before. Every physical component fails at some rate. Aircraft, power
grids and medical devices are still reliable because their designers assume failure and build
controls around it:

- **Redundancy.** More than one independent check on the same property.
- **Feedback loops.** Measure the output, compare it with the goal, correct.
- **Independent verification.** The party that checks is not the party that built.
- **Failsafes.** When a check fails, the system stops in a safe state instead of carrying on.

Apply the same thinking to agents. Keep a healthy mistrust of stochastic output. Trust evidence,
not claims. A harness is the set of controls that turns an unreliable component into a system you
can rely on.

## 2. The V-model

The V-model is a systems-engineering lifecycle. Read it as a letter V.

```
  Validation (acceptance) ........................ Concept of operations
     System / UI verification .................. Requirements + test plan
        Integration verification ............. Design
           Unit verification ............... Specifications
                         \               /
                          \   Build    /
```

- **Left side: decomposition.** Start from the need and break it down. System understanding,
  concept of operations, requirements, design, specifications, plan. Each level is more concrete
  than the one above.
- **Bottom: build.** Write the code against the specifications.
- **Right side: integration and verification.** Climb back up. Each level on the right verifies
  its paired level on the left. Unit tests verify specifications. Integration tests verify design.
  System and UI tests verify requirements. Acceptance validates the concept of operations.
- **Top right: validation.** A human confirms the right thing was built for the real users.

The key idea is pairing. You write the check for a level at the same time you define that level.
That is why the test plan is written with the requirements, before any code.

## 3. Verification is not validation

- **Verification:** did we build it right? Does the output meet the specification?
- **Validation:** did we build the right thing? Does it solve the user's real problem?

Verifiability is high at the bottom of the V and low at the top. A unit test is deterministic and
cheap. "Is this the feature the support team actually needed?" is neither.

Design rule: automate hard at the bottom. Concentrate human attention at the top, where
verifiability is low. A harness that asks a human to review every line of code wastes the human.
A harness that never asks a human whether the plan is right will build the wrong thing very
efficiently.

## 4. Deterministic gates beat prompts

A prompt is a request. A gate is a wall.

If the agent keeps skipping tests even though the prompt says "always run the tests", writing the
instruction in capitals will not fix it. Put the tests in a gate the agent cannot argue with: a
script that fails the stage, a required CI check, a pre-commit hook, a workflow step that will not
start until the previous step returns success.

Rules:
- Every check that can be deterministic should be deterministic.
- The order of stages is held by a script, a workflow or CI, not by instructions alone.
- A gate's result is its exit code and output, not the agent's summary of it.
- A gate the agent can edit is not a gate. Protect gate definitions with write fences.

## 5. Independent adversarial verification

Some properties cannot be checked by a script. "Does every requirement trace to a use case?" "Is
this design consistent with the existing patterns?" For these, use an independent verifier:

- **Fresh context.** A new agent session with no memory of the generation.
- **Artefact only.** It reads the artefacts, the repo and the ticket. It never reads the
  generator's transcript or reasoning, so it cannot be talked into agreement.
- **Adversarial stance.** Its job is to find failures, not to approve.
- **Explicit gates.** A numbered checklist. Every gate must pass. No partial credit.
- **No accept-with-followups.** A failed gate is a rejection. Strength in one area never offsets
  a failure in another.
- **Escalation.** After a set number of consecutive rejections (three is a good default), stop and
  put the disagreement to a human.

The same idea applies at the end: a completion verifier checks that every requested deliverable
has real evidence before anyone says "done". See `verifiers.md`.

## 6. Requirements traceability

Traceability is the thread that ties the V together:

use case -> requirement -> design element / specification -> plan task -> test -> evidence

Two rules close the thread:
- **No orphans.** Every requirement traces up to a use case. Every task and spec traces to a
  requirement. Code with no requirement is scope creep.
- **No untested requirements.** Every requirement has at least one test at the right level, and
  at handback every test has a result.

See `rtm.md` for the format.

## 7. Humans stay in the loop at explicit gates

Two human gates are the minimum:

1. **Plan approval.** After the adversarial reviewer accepts the definition, and before any code.
   The human checks the plan matches the intent.
2. **Handback and acceptance.** After verification is green and evidence is attached. The human
   validates the result.

Approval must be explicit and affirmative. A negation never approves. The agent's own comments,
labels or status changes never approve. A correction to scope voids earlier approval and sends the
work back to definition.

Add more human gates only where the risk demands it (for example a security sign-off). Every extra
stop has a cost.

## 8. The skip ledger: never skip silently

Some tickets do not need the full ceremony. That is fine, if it is visible.

- **Propose.** List each step to skip or condense, with a reason, and ask for approval.
- **Disclose.** Record what was skipped, why, and who approved it, in a durable place on the
  ticket. If nothing was skipped, say so.
- **Never skippable.** Testing, regression and evidence. These are the controls that catch the
  unreliable component.

## 9. Honest limitations

- **It is slower.** Definition, review and verification cost time and tokens. That is the price of
  evidence.
- **Top-of-V validation stays hard.** No automated check fully answers "did we build the right
  thing". A human still has to look.
- **Pieces alone are not the V.** TDD alone verifies code against tests the same agent wrote.
  Adversarial review alone checks documents but not behaviour. The value comes from the pairing
  and the traceability between levels.
- **A gate you cannot run is a known gap.** Say so. Do not count it as a pass.
- **The harness needs tuning.** The first version will be wrong in places. Feed every miss back
  into `AGENTS.md` and the harness.

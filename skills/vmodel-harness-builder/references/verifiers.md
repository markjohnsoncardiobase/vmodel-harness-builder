# Verifiers

A verifier is an agent whose only job is to find reasons the work is not acceptable. The harness
uses two:

1. **The adversarial definition reviewer.** Checks the left side of the V before a human is asked
   to approve the plan.
2. **The completion verifier.** Checks the right side before anyone says "done".

Both follow the same design rules.

## Design rules

1. **Fresh context.** Start a new agent session or subagent. It has no memory of the generation.
2. **Artefact only.** It reads the artefacts, the repo, the ticket and the test output. It never
   reads the generator's transcript, scratch notes or reasoning. A verifier that sees the
   argument for a decision tends to agree with it.
3. **Strongest model available.** Verification is where the strong model pays off. Use a
   different model family as well if you have one; correlated errors are the enemy of redundancy.
4. **Adversarial stance.** Its instructions say its job is to find failures. Approval is the
   result when it cannot find any.
5. **Explicit numbered gates.** A fixed checklist written in the harness. No free-form "does this
   look OK".
6. **Every gate must pass.** One FAIL rejects the whole package. Strength in one area never
   offsets a failure in another.
7. **No accept-with-followups.** "Pass, but fix X later" is a FAIL.
8. **Evidence required.** Every FAIL cites the artefact and the repo, ticket or test evidence.
   Every PASS on a factual claim (a file exists, a test passed) is spot-checked, not assumed.
9. **Whole-package repair.** On rejection, the generator repairs the whole package, not only the
   failed file, because a fix must flow through traceability. Then a fresh verifier checks again.
10. **Escalation.** After three consecutive rejections, stop. Put both positions to a human.
11. **Read-only.** The verifier never edits the artefacts or code it is judging.

## Adversarial definition reviewer

### Default gates

1. Triage path justified; risk areas identified; draft skip ledger present; testing, regression
   and evidence not skipped.
2. System understanding matches the repo. Spot-check that named files, components and
   conventions exist.
3. ConOps complete: all actors, numbered use cases including edge and exception cases,
   permissions impact.
4. Requirements have stable IDs, are atomic and testable, and functional ones trace to use cases.
5. Test plan covers every requirement at every applicable level, including named existing
   regression suites and UI or end-to-end evidence.
6. Design addresses data, contract and migration impact, reuses existing patterns, records
   decisions with rejected alternatives.
7. Specifications are implementation-ready and each traces to a requirement ID.
8. Research is verified against real code: no phantom APIs, no unexplained pattern conflicts,
   every user-supplied reference read.
9. Plan has one task per build item plus a testing task, requirement mappings, dependencies, and
   a declared change scope with protected paths.
10. Traceability closes: no orphan requirements, no untested requirements.
11. Skip disclosure present and consistent with the path and any approvals.

Add stack- or domain-specific gates from the interview (for example "every new endpoint has an
authorisation requirement and test", "every schema change has a rollback plan").

### Prompt template

```
You are an independent adversarial reviewer. Your job is to find reasons this definition package
is NOT ready for human approval. You have a fresh context. Do not ask for or read any transcript
of how the package was produced.

Inputs:
- Ticket: {{TICKET_REFERENCE}} (read the description, all comments and every attachment)
- Artefacts: {{ARTEFACT_PATHS}}
- Repository: the current working tree
- Binding context: {{AGENTS_FILE}}

Check every gate below. For each gate output PASS or FAIL. For every FAIL, cite the artefact and
section, and the repo, ticket or attachment evidence. Spot-check factual claims by opening the
files they name. There is no partial pass and no "pass with follow-ups". Any FAIL means REJECT.

Gates:
{{NUMBERED_GATES}}

Output format:
VERDICT: ACCEPT | REJECT
GATE 1: PASS | FAIL - <finding and evidence>
...
GATE N: PASS | FAIL - <finding and evidence>
REPAIRS REQUIRED: <list, empty if ACCEPT>

Do not edit any file.
```

## Completion verifier

The completion verifier checks that the work requested is the work delivered, with evidence. It
stops the common failure where an agent reports success it has not earned.

### What it checks

1. **The ledger of deliverables.** List every atomic deliverable from the ticket and the approved
   plan, each with a stable ID and acceptance criteria. Planning, building and testing are
   separate deliverables when all three were requested. Do not drop the hard ones.
2. **Evidence per deliverable.** Real evidence only: test output, exit codes, recordings, the
   read-back of an external write (comment posted, file attached). A file path without inspected
   contents is not evidence. A promise is not evidence. A failed command is not evidence.
3. **The RTM closes.** Every requirement has tests, and every test has a passing result from this
   run, linked to evidence.
4. **Per-deliverable verdict.** Each deliverable gets complete, partial or missing, with a
   confidence. Totals are counted from the per-deliverable verdicts, never guessed.
5. **Coverage.** The ledger represents the whole request. Nothing requested is missing from it.

### Repair loop

- If any deliverable is not complete, return the full list of unfinished items to the builder,
  each with: ID, original request, acceptance criteria, status, work done, evidence, the precise
  gap, and any blocker.
- The repair is an instruction to keep working, not text to paste to the user. The builder acts,
  re-runs the checks and updates the evidence.
- Re-verify the whole ledger, not only the repaired items.
- Limit automatic repair to three passes. If there is no progress, a missing approval, or a real
  blocker, stop and report the remaining items honestly. Only a full pass means done.
- Preserve completed work. Do not repeat external side effects that were already verified.

### Prompt template

```
You are an independent completion verifier. Your job is to decide, from evidence only, whether
every requested deliverable is complete. You have a fresh context. Do not read the builder's
transcript.

Inputs:
- Ticket and approved plan: {{TICKET_REFERENCE}}, {{PLAN_PATH}}
- Traceability matrix: {{RTM_PATH}}
- Test and gate output from this run: {{GATE_OUTPUT_PATHS}}
- Evidence: {{EVIDENCE_PATHS}}
- Candidate handback text: {{HANDBACK_DRAFT}}

Steps:
1. Build the deliverable ledger from the ticket and plan. Each item: ID, description,
   acceptance criteria.
2. For each item, find real evidence. Open the files. Read the test output. Check exit codes.
   Paths without contents, promises and failed commands do not count.
3. Check every RTM row has a passing result from this run.
4. Check every claim in the candidate handback is backed by evidence.

Output format:
VERDICT: PASS | REPAIR | BLOCKED
REQUESTED: <count of ledger items>
COMPLETE: <count with complete verdict>
ITEM <id>: complete | partial | missing - confidence <high|medium|low> - <evidence or gap>
UNSUPPORTED CLAIMS IN HANDBACK: <list>
REPAIR LIST: <every non-complete item with its criteria, evidence so far, and precise gap>

Do not edit any file.
```

## Privacy and safety

- Never send secrets, credentials, personal data or raw transcripts to an external verifier
  service. Abstract the evidence or report the verification as blocked.
- Verification does not replace human approval for actions that leave the machine (sending,
  posting, deploying). Those keep their own approval rules.

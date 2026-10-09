# Workflow outline (optional orchestration)

If the user's tool can run a script or workflow that spawns agents (for example Claude Code
workflows, an agent SDK script, or a CI pipeline that calls an agent), generate one so the stage
order is held by code instead of by instructions. The stage definitions in the harness skill stay
identical either way.

Pseudocode, language-neutral:

```
ticket = intake.fetch(ticket_id)            # read description, comments, every attachment

rejections = 0
loop:
    package = spawn(generator, model=STRONG, fresh=True, inputs=[ticket, repo, agents_file])
    verdict = spawn(reviewer, model=STRONGEST, fresh=True,
                    inputs=[package.paths, ticket, repo])      # never the generator transcript
    if verdict.accept: break
    rejections += 1
    if rejections == 3: escalate_to_human(verdict, package); stop
    feed verdict.repairs to generator (whole-package repair)

store(package); post(plan_digest)
wait_for_human_approval()                   # hard stop; corrections restart the loop

require(run(BASELINE_COMMAND).ok)
for group in plan.parallel_groups():
    parallel(spawn(builder, model=BUILD, task=t) for t in group)
    require(run(PRECOMMIT_COMMANDS).ok) before each commit

loop:
    results = [run(UNIT), run(INTEGRATION), run(REGRESSION), run(E2E, record=True), scope_check()]
    if all ok: break
    send failures back to builders

for attempt in 1..3:
    v = spawn(completion_verifier, model=STRONGEST, fresh=True,
              inputs=[ticket, plan, rtm, gate_outputs, evidence, handback_draft])
    if v.pass: break
    act on v.repair_list; re-run affected gates
else:
    report remaining items honestly; stop

open_pr(); post(handback)                   # only here
wait_for_human_acceptance()
```

Rules for the script:
- A stage cannot start until the previous gate returns success.
- Gate results come from exit codes, not from an agent's summary.
- Keep intermediate artefacts in workflow state or files, not in the main conversation.
- The PR call appears in exactly one place: after completion verification passes.

# Interview question bank

Use this in builder step 4. Ask three to five questions per batch. Before each question, check the
research findings from step 3. If research answered it, state what you found and ask for a yes
instead of asking the question cold.

Each question lists why it matters and how the answer changes the generated harness. Keep a
running answer sheet with one line per question. Read it back at the end.

## A. Intake

**A1. Where does work come from?** Jira, GitHub Issues, Linear, Azure Boards, GitLab, Slack
threads, email, a spreadsheet, or plain prompts in the terminal.
- Why: the ticket is the anchor for every artefact, comment and approval.
- Changes: the intake section of the harness, the tools it calls, and where approvals are read.
  For plain prompts, the harness creates a lightweight record (an issue, or a markdown file in the
  repo) so the work is still anchored.

**A2. How will the agent read and write it?** CLI (`gh`, `glab`, `acli`, `linear`), an MCP server,
a REST API with a token, or manual copy and paste.
- Why: a harness that cannot post comments or read approvals cannot hold the human gates.
- Changes: the exact commands for fetch, comment, attach, status change. If write access is not
  available, the harness writes artefacts to the repo and the human posts the digest.

**A3. What statuses or labels does a ticket move through?**
- Why: status changes make progress visible without reading code.
- Changes: the status map in the harness (for example: defining, awaiting answers, ready for
  approval, approved, building, done). Only move a status after the phase's work is complete.

**A4. Can tickets carry attachments, and does the agent have to read them?** Specs, designs,
screenshots, logs.
- Why: missed attachments are a common source of wrong builds.
- Changes: a hard rule to read every attachment in full before definition, and a review gate that
  checks it.

**A5. Is there a mapping from repo to project or board?**
- Why: the agent should know where to raise a new ticket without asking.
- Changes: a mapping table in the intake section.

## B. What is being built

**B1. What is the product and who uses it?** Internal tool, public web app, mobile app, API,
library, data pipeline, ML service, infrastructure.
- Why: the type of product decides which verification levels apply.
- Changes: the right side of the V. A library has no UI journey. A data pipeline needs data
  checks. An ML feature needs evals.

**B2. Who are the end users and what happens to them if a change is wrong?**
- Why: this sets the risk floor and where humans must look.
- Changes: triage rules and the never-lightweight list.

**B3. Which parts of the system are fragile, critical or regulated?**
- Why: these become protected paths and forced full-path triggers.
- Changes: write fences, triage rules, extra human gates if needed.

**B4. Confirm the research summary.** Present the stack, commands and risky areas found in step 3.
- Why: the harness must use real commands.
- Changes: the gate table.

## C. Nature of the tickets

**C1. What is the mix of bugs, features, refactors and chores?**
- Why: bugs need a reproduction step and a regression test first; features need ConOps and
  requirements.
- Changes: a bug variant of the definition (reproduce, root cause, failing test first).

**C2. How big is a typical ticket?** Hours, days, weeks.
- Why: big tickets need decomposition into subsystems or child tickets before the V runs.
- Changes: a size rule in triage that splits large work.

**C3. How well written are tickets?** Clear acceptance criteria, or one line and a screenshot?
- Why: thin tickets need more clarification and enrichment.
- Changes: the clarification loop (batched questions, once per iteration, mirrored on the ticket)
  and the description enrichment rule (append a clearly marked agent section, never rewrite the
  original).

**C4. How ambiguous is the intent usually?**
- Why: high ambiguity raises the value of ConOps and the plan approval gate.
- Changes: how much of the left side the lightweight path may condense.

## D. Risk profile

**D1. What is the worst realistic outcome of a bad change?** Cosmetic glitch, lost revenue, data
loss, security breach, legal or regulatory breach, physical harm.
- Why: risk decides ceremony.
- Changes: triage thresholds, extra human gates, mandatory security scans.

**D2. How reversible are changes?** Feature flags, instant rollback, or one-way migrations?
- Why: irreversible changes need more verification before merge.
- Changes: migration rules and a forced full path for one-way changes.

**D3. Are there compliance requirements?** Audit trail, change control, separation of duties.
- Why: the ticket history may be your audit evidence.
- Changes: artefact retention, approval wording, who may approve.

## E. Verification available

**E1. Which of these exist and run today?** Unit, integration, contract, regression, UI or
end-to-end (with recording), performance, accessibility, security scans (SAST, dependency audit,
secret scan), data checks (schema, row counts, data contracts, dbt tests), evals for model output,
manual checks.
- Why: each level of the right side needs a real check.
- Changes: the gate table. Anything missing goes in "Known gaps" with a suggestion.

**E2. Do the suites pass today?** Any known-flaky tests?
- Why: a red baseline makes every gate meaningless.
- Changes: a baseline step before build, and a named flaky list.

**E3. How is the app run locally, and where are test credentials and fixtures?**
- Why: UI verification must run against the real app without asking for secrets.
- Changes: the run command and credential location in the harness and `AGENTS.md`.

**E4. What evidence would convince a reviewer?** Video, trace, screenshots, query output, metrics.
- Why: evidence is what makes handback trustworthy.
- Changes: the evidence rule for the UI or end-to-end level.

**E5. What runs in CI, and which checks are required?**
- Why: CI is the gate the agent cannot bypass.
- Changes: mirror local gates in CI, or recommend adding them.

## F. Deterministic, adversarial or human

**F1. For each check, can a script decide it?** If yes, make it deterministic.
- Changes: gate type D.

**F2. What needs judgement but not a human?** Traceability completeness, design consistency,
requirement quality, comment hygiene.
- Changes: gate type A, added to the adversarial reviewer's checklist.

**F3. What must a human decide?** Plan intent, acceptance, security sign-off, release.
- Changes: gate type H. Keep these few.

## G. Humans in the loop

**G1. Who approves the plan and who accepts the handback?** Same person, or different?
- Changes: who is mentioned on the digest and handback, and whose reply counts.

**G2. What counts as approval?** A keyword (`APPROVE-PLAN`), a label set by a human, a PR review.
- Changes: the approval rule. Always: affirmative, newer than the digest, from a human.

**G3. How quickly do humans respond?**
- Why: slow approvals favour batching questions and fewer stops.
- Changes: the clarification batching rule.

## H. Budget and tolerance

**H1. Which models and tools do you have access to?**
- Changes: the role-to-model map. Strongest model for the definition generator and every
  verifier. A cheaper model may build and run tests.

**H2. What token or cost budget per ticket is acceptable?**
- Changes: how much runs in parallel, and lightweight path thresholds.

**H3. How much slower can delivery be?**
- Why: the V costs time. Set expectations now.
- Changes: triage thresholds and what the lightweight path condenses.

**H4. Does your tool support orchestration (workflows, scripts, CI pipelines for agents)?**
- Changes: whether the harness ships a workflow script that holds the stage order in code.

## I. Triage paths

**I1. Which paths do you want?** Default: lightweight, standard, full.
- Changes: path definitions in the harness.

**I2. What forces the full path?** Risk areas, migrations, public API changes, size.
- Changes: triage rules.

**I3. What may the lightweight path condense?** Default: one combined `definition.md` and a short
adversarial pass. Never testing, regression or evidence.
- Changes: the lightweight section.

## J. Artefacts and handback

**J1. Where should artefacts live?** Ticket attachments, a repo folder (for example
`docs/changes/<ticket>/`), the PR description, or several.
- Changes: artefact paths and attach commands.

**J2. What should the handback contain and where?**
- Default order: verification summary, evidence, skip disclosure, test data notes, PR link.
- Changes: the handback template.

**J3. Does the agent open PRs, merge, or deploy?**
- Default: open a ready (not draft) PR at handback only. Never merge or deploy unless the team
  explicitly assigns that role.
- Changes: handback stage boundaries.

## K. Memory and improvement

**K1. Is there an `AGENTS.md`, `CLAUDE.md` or similar file?** Who maintains it?
- Changes: where durable learnings go and the entry format.

**K2. What should always be recorded?** Run commands, credentials location, decisions, gotchas,
flaky tests.
- Changes: the continual-improvement rules.

## L. Factory only

Ask these only if the user chose factory mode in step 2.

**L1. Where will the factory run?** A VM, a container platform, a CI scheduler, a spare machine.
- Changes: the runner entry point, how secrets reach it, and how it is restarted.

**L2. What starts a pass?** A schedule (every N minutes), a tracker webhook, a label or status change.
- Changes: the trigger, and how the factory picks which ticket to work on next.

**L3. Where does state live?** A state file, a small database, the ticket itself.
- Changes: how passes stay idempotent and resume after a crash.

**L4. How many tickets at once?** One global worker, one per ticket owner, one per product.
- Changes: the locking rule.

**L5. How does each ticket get a clean world?** A worktree per ticket, a fresh copy of the app and
database, a preview environment from CI.
- Changes: the setup and teardown scripts and where reviewers open the live app.

**L6. Who does the factory post as?** A bot or service account, never a person.
- Changes: the identity used for comments, so the factory never mistakes its own comment for a
  human approval.

**L7. How is approval detected?** A standalone approval comment from a named human, newer than the
plan digest.
- Changes: the approval rule. Negations and bot comments never approve.

**L8. When should it give up and ask?** After N rejections in a row, after a time budget, on a
blocked environment.
- Changes: escalation and who gets notified.

**L9. Where do people hear from it?** Chat channel, email, the ticket only.
- Changes: notification hooks.

**L10. What are the usage limits?** Model subscriptions, API budgets, CI minutes.
- Changes: limit guards and any provider fallback.

**L11. How are changes to the factory itself released?** Straight to main, or a probation release
that rolls back on failure.
- Changes: the release process for the harness code.

## Closing the interview

1. Read the answer sheet back as a short list.
2. Flag every level of the V that has no real verification yet.
3. Confirm the proving-run ticket for step 7.
4. Ask for a yes, then move to builder step 5.

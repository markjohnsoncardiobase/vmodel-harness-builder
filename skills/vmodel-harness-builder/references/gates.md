# Gates

A gate is a check that must pass before the next stage starts. Each gate has a type, an exact
check, and a defined failure action. Use this file in builder step 4 to type every gate in the
user's harness.

## Three gate types

| Type | Decided by | Strength | Use for |
| --- | --- | --- | --- |
| **D** Deterministic | A command's exit code and output | Strongest. Cannot be argued with. | Anything a script can decide. |
| **A** Adversarial | An independent fresh-context agent against a numbered checklist | Medium. Catches judgement failures a script cannot see. | Completeness, consistency, traceability, quality of reasoning. |
| **H** Human | An explicit affirmative reply from a named person | Essential where verifiability is low. Expensive. | Intent, acceptance, high-risk sign-off. |

Order of preference: D, then A, then H. Move a check to D whenever you can. Keep H gates few and
place them where they add the most: plan approval and handback.

## Rules for deterministic gates

- The gate result is the exit code and output. Never the agent's summary.
- Run the same command locally and in CI. CI is the gate the agent cannot bypass.
- Protect gate definitions (test config, CI files, lint rules, coverage thresholds) with write
  fences. An agent that can edit the gate can pass it trivially.
- Do not allow skipped, disabled or deleted tests to make a suite green. Add a check that counts
  skips and compares test totals with the baseline if the stack allows it.
- Record a baseline before build. A red baseline must be fixed or explicitly scoped before work
  starts.

## Rules for adversarial gates

See `verifiers.md`. Short version: fresh context, artefact only, explicit numbered gates, every
gate must pass, evidence for every FAIL, escalate after three consecutive rejections.

## Rules for human gates

- Approval is affirmative and standalone. A negation never approves.
- Only a human reply newer than the request counts. The agent's own comments, labels or statuses
  never approve.
- A correction voids earlier approval. Return to definition, repair, re-review, re-request.

## Standard gate set for a generated harness

| # | Gate | Type | Stage | Blocks |
| --- | --- | --- | --- | --- |
| G1 | Ticket anchored, every attachment read | A | Intake | Definition |
| G2 | Definition package passes every review gate | A | Definition | Plan approval request |
| G3 | Plan approved by a human | H | Definition | Build |
| G4 | Baseline suite green (or scoped) | D | Before build | Build |
| G5 | Per-commit: tests, typecheck, lint pass | D | Build | Commit |
| G6 | Diff stays inside declared change scope | D | Build and verification | Verification |
| G7 | Unit suite green | D | Verification | Integration |
| G8 | Integration and contract suites green | D | Verification | Regression |
| G9 | Regression suites for affected area green | D | Verification | UI |
| G10 | UI or end-to-end green with evidence attached | D plus evidence | Verification | Completion |
| G11 | Every RTM row has a passing result and evidence | D plus A | Completion | Handback |
| G12 | Comment and hygiene audit of the diff | A (or D with a linter) | Completion | Handback |
| G13 | Skip disclosure present and consistent | A | Completion | Handback |
| G14 | Handback accepted by a human | H | Handback | Close |

## Menu of deterministic gates by stack

Use only commands that exist in the user's repo or that the user confirms. These are examples to
prompt the conversation, not defaults to copy.

### JavaScript and TypeScript
- Typecheck: `tsc --noEmit`
- Lint and format: ESLint, Biome, Prettier check
- Unit: Vitest, Jest, node test runner
- Integration: Supertest against the API, Testcontainers for databases
- Contract: Pact, OpenAPI schema validation, Zod schema tests
- UI and end-to-end: Playwright with `video: 'on'` and trace, Cypress with video
- Accessibility: axe via Playwright
- Security: `npm audit` or equivalent, Semgrep, secret scanning

### Python
- Typecheck: mypy, Pyright
- Lint and format: Ruff, Black check
- Unit and integration: pytest, pytest-cov with a threshold
- Contract: Pydantic or JSON Schema validation, Schemathesis for APIs
- Data: dbt tests, data contracts, Great Expectations or Pandera, row-count and null checks
- Security: Bandit, pip-audit, secret scanning

### JVM (Java, Kotlin)
- Build and test: Gradle or Maven test tasks, JUnit
- Static analysis: SpotBugs, Detekt, Checkstyle
- Integration: Testcontainers
- Contract: Spring Cloud Contract, Pact

### .NET
- Build and test: `dotnet build`, `dotnet test`
- Analysis: Roslyn analysers with warnings as errors
- UI: Playwright for .NET

### Go
- `go vet`, `staticcheck`, `go test ./... -race`

### Mobile
- iOS: `xcodebuild test`, XCUITest with screen recording
- Android: Gradle unit tests, Espresso, emulator screen recording

### Infrastructure as code
- `terraform validate`, `terraform plan` with a reviewed plan file, tflint, Checkov

### Databases and migrations
- Migration applies cleanly on a fresh database and on a copy of the current schema
- Migration rolls back cleanly (where the tool supports down migrations)
- Schema diff matches the design

### ML and LLM features
- Offline evals with a fixed dataset and a pass threshold
- Regression evals against the previous model or prompt version
- Output schema validation

### Repository hygiene
- Diff-scope check: compare changed paths against the plan's allowed and protected paths
- Skipped-test count did not rise
- No secrets in the diff (gitleaks, trufflehog)
- Commit messages reference the ticket and task ID

## Writing a gate in the generated harness

Each gate entry in the generated skill should state:

```
G<n>  <name>
Type: D | A | H
Check: <exact command, checklist reference, or approval rule>
Pass when: <exit code 0 / every item PASS / affirmative human reply newer than request>
On fail: <return to stage X; never proceed; post blocked comment if it cannot run>
```

If a gate cannot run (missing tool, broken environment), fail loudly. Post what was tried and what
is missing. Never downgrade a gate to make progress.

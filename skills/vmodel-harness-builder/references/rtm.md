# Requirements traceability matrix (RTM)

The RTM is the thread that ties the left side of the V to the right side. It answers two
questions at any moment: "why does this code exist?" and "how do we know this requirement works?".

## IDs

Use short, stable IDs. Never renumber after the plan is approved; retire IDs instead.

| Prefix | Meaning | Example |
| --- | --- | --- |
| `UC-` | Use case (from ConOps) | `UC-03` |
| `BR-` | Business requirement | `BR-01` |
| `FR-` | Functional requirement | `FR-07` |
| `NFR-` | Non-functional requirement (performance, security, accessibility) | `NFR-02` |
| `D-` | Design element or decision | `D-04` |
| `SP-` | Specification item | `SP-11` |
| `T-` | Plan task | `T-05` |
| `UT-` / `IT-` / `RT-` / `E2E-` | Unit, integration, regression, UI or end-to-end test | `UT-14` |

## Format

Keep the matrix in `traceability.md` as a table. One row per requirement. Use lists inside cells
when a requirement maps to several items.

| Req | Traces to | Design / spec | Task | Planned tests | Result | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| FR-01 | UC-01 | D-01, SP-01 | T-01 | UT-01, UT-02, IT-01, E2E-01 | | |
| FR-02 | UC-01, UC-02 | D-02, SP-02, SP-03 | T-02 | UT-03, IT-02, RT-01, E2E-02 | | |
| NFR-01 | BR-01 | D-03 | T-03 | IT-03 (p95 under 300 ms) | | |

The **Result** and **Evidence** columns stay empty during definition. The completion verifier fills
them from this run's gate output: `PASS` or `FAIL`, and a link or path to the output, recording or
trace.

## Rules

1. **No orphan requirements.** Every `FR-` traces to at least one `UC-`. Every `NFR-` traces to a
   `BR-` or a stated constraint.
2. **No orphan work.** Every `SP-` and `T-` traces to at least one requirement. Work with no
   requirement is scope creep: add a requirement (and re-approve) or drop the work.
3. **No untested requirements.** Every requirement has at least one planned test at the level that
   verifies it (see the pairings in `v-model.md`).
4. **Regression is named.** At least one row, or a footer line, names the existing suites run for
   regression.
5. **Closed at handback.** Every row has a `PASS` result from this run and an evidence link. A
   `FAIL` or an empty cell blocks handback.
6. **Changes flow through.** When any artefact changes, update the matrix in the same repair pass.

## Deterministic check (optional, recommended)

If the harness can run a script, add a small RTM linter. It parses `traceability.md` and fails if:

- any requirement in `requirements.md` is missing from the matrix;
- any matrix row has no planned tests;
- any task in `IMPLEMENTATION_PLAN.md` maps to no requirement;
- at handback, any row lacks a `PASS` result or evidence.

This moves part of adversarial gate 10 from **A** to **D**.

## Closing the thread

The verification summary at handback is a view of the closed matrix:

```
FR-01  UT-01 PASS, UT-02 PASS, IT-01 PASS, E2E-01 PASS  evidence: <link>
FR-02  UT-03 PASS, IT-02 PASS, RT-01 PASS, E2E-02 PASS  evidence: <link>
Regression: <suite names> PASS (<n> tests, 0 skipped added)
```

A reader should be able to start from any requirement and reach its evidence, or start from any
changed file and reach the requirement that justified it.

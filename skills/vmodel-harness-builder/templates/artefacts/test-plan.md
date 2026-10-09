# Test plan: {{TICKET}}

Written before any code. Every requirement has tests at each applicable level.

## Unit

| Test ID | Requirement | Case | Expected |
| --- | --- | --- | --- |
| UT-01 | FR-01 | success | |
| UT-02 | FR-01 | empty or null input | |
| UT-03 | FR-01 | zero / negative / out-of-bounds | |

## Integration and contract

| Test ID | Requirement | Boundary | Direction | Expected |
| --- | --- | --- | --- | --- |
| IT-01 | FR-01 | API to database | write then read back | |
| IT-02 | FR-01 | database to API | read path | |

## Regression

| Test ID | Existing suite | Why it could be affected |
| --- | --- | --- |
| RT-01 | | |

Known flaky tests in these suites:

## UI or end-to-end

| Test ID | Requirement | Journey | Evidence |
| --- | --- | --- | --- |
| E2E-01 | FR-01 | | recording / trace / output |

## Other checks
Performance, accessibility, security scans, data checks, evals. Write "not applicable" with a reason
where they do not apply.

## Test data
Fixtures, seeds and accounts needed. Reference where credentials live; never write them here.

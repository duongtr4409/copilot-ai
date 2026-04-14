---
name: automated-quality-gate
description: "Cross-check implementation against BA acceptance criteria, detect missing AC coverage, and generate edge-case unit tests with a clear pass/conditional/fail verdict. Use when doing BA-to-Code alignment, requirement coverage audits, and unit test hardening before merge."
argument-hint: "BA requirements/AC, target code scope (files/modules/diff), language/test framework, and coverage constraints"
user-invocable: true
---

# Automated Quality Gate (Cong chat luong tu dong)

## What This Skill Produces
- BA-to-Code alignment report with Covered/Partial/Missing status per acceptance criterion.
- Evidence mapping from requirement to implementation location.
- Gap analysis with risk level and minimal remediation guidance.
- Edge-case-focused unit test matrix and ready-to-run test cases.
- Final gate verdict: PASS, CONDITIONAL PASS, or FAIL.

## When To Use
- Before pull request merge to verify business requirement coverage.
- During QA handoff when BA/PA asks for requirement traceability.
- When refactoring high-risk modules that may drop hidden AC behavior.
- When test coverage exists but edge-case robustness is uncertain.

## Required Inputs
- BA requirements: user story, BRD/FSD extract, or explicit acceptance criteria list.
- Code scope: selected code, file list, module, or PR diff.
- Constraints: language, test framework, naming style, and coverage goals.
- Optional: risk priorities (security, data integrity, performance, compliance).

If AC IDs are missing, generate stable IDs: AC-01, AC-02, AC-03...
If requirements are ambiguous, list assumptions first, then continue analysis.

## Workflow

### Step 1: Normalize Requirements
1. Parse BA requirements into atomic AC statements (one behavior per AC).
2. Add IDs if missing.
3. Mark each AC with business criticality: Critical, High, Medium, Low.

Quality check:
- Every AC is testable and unambiguous.
- Duplicated or overlapping AC entries are merged and noted.

### Step 2: BA-to-Code Alignment
1. Inspect the target code scope.
2. Map each AC to implementation evidence (function, class, endpoint, guard, validation).
3. Assign coverage status:
- Covered: AC behavior is fully implemented.
- Partial: some paths implemented, edge or negative path missing.
- Missing: no implementation evidence found.
4. Record evidence as file + line references.

Decision branch:
- If any Critical AC is Missing: immediately set provisional verdict to FAIL.
- If only Medium/Low AC are Partial: provisional verdict can remain CONDITIONAL PASS.

### Step 3: Gap and Risk Diagnosis
1. For each Partial/Missing AC, describe failure impact.
2. Tag risk type where relevant:
- Business rule violation
- Data integrity risk
- Security exposure
- User experience regression
3. Propose minimal remediation steps that preserve existing architecture.

Decision branch:
- If remediation requires behavior change in production code, mark as "Code change required".
- If issue can be validated via tests only, mark as "Test-only hardening".

### Step 4: Edge-Case Unit Test Generation
1. Build a test matrix for each high-risk function/module.
2. Include boundary and negative scenarios:
- boundary values
- null/empty input
- invalid format/type
- permission/role denial
- dependency failure/timeouts
- concurrency/race conditions (if applicable)
3. Generate ready-to-run unit tests using repository conventions.
4. Ensure deterministic setup/teardown and descriptive test names.

Quality check:
- Every Missing/Partial AC has at least one test case proposal.
- Critical AC has both positive and negative path tests.

### Step 5: Final Quality Gate Verdict
Use this rule set:
- PASS:
  - No Critical AC is Missing.
  - No High AC is Missing.
  - Edge tests cover all critical logic branches.
- CONDITIONAL PASS:
  - No Critical AC is Missing.
  - Some High/Medium AC are Partial with low operational risk.
  - Clear remediation plan exists.
- FAIL:
  - Any Critical AC is Missing, or
  - Critical edge scenarios are untested, or
  - Business behavior can break under normal invalid input.

## Output Contract
Return results in this structure:

1. Analysis Scope and Assumptions
2. BA-to-Code Alignment Table
3. Gaps and Risk Assessment
4. Edge-Case Unit Test Matrix
5. Generated Unit Tests
6. Final Verdict and Next Actions

### BA-to-Code Alignment Table
| AC ID | Requirement Summary | Status (Covered/Partial/Missing) | Evidence (file + line) | Criticality | Risk |
|---|---|---|---|---|---|

### Edge-Case Unit Test Matrix
| Target Module/Function | Scenario Type | Test Case | Expected Result | Priority |
|---|---|---|---|---|

## Completion Criteria
- All AC are mapped to code evidence or explicitly marked Missing.
- Every Missing/Partial AC has remediation guidance.
- Edge-case unit tests are generated for all Critical and High-risk behavior.
- Verdict rationale is explicit and reproducible.

## Guardrails
- Do not change production code unless explicitly requested.
- Do not claim coverage without concrete evidence.
- If repository test framework is unclear, state assumption and provide framework-specific variant.

## Example Invocations
- /automated-quality-gate Validate checkout module against BA AC and generate edge-case unit tests in Jest.
- /automated-quality-gate Audit auth service for missing acceptance criteria and output PASS/FAIL verdict.
- /automated-quality-gate Compare PR diff with user-story AC and produce remediation plan for gaps.

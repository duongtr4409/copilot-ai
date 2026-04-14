---
description: "Use when Code Reviewer performs code review: severity classification, SOLID/DRY analysis, security surface inspection, blueprint compliance, test coverage validation, and merge gate decisions with actionable evidence."
name: "Code Reviewer Quality Engineering"
---

# Code Reviewer Quality Engineering Instruction

## Table of Contents
1. Review Priority Hierarchy
2. Severity Classification Guide
3. Security Review Checklist
4. Blueprint Compliance Check Protocol
5. Test Coverage Requirements
6. Merge Gate Decision Tree
7. Rework Request Protocol
8. Review Workflow Steps
9. Output Quality Gate

## 1. Review Priority Hierarchy
Review findings MUST be evaluated in this priority order:

1. **Correctness** (P0): Logic errors, wrong behavior, data corruption risks.
2. **Security** (P1): Injection, authentication/authorization flaws, secrets exposure.
3. **Data Integrity** (P2): Transaction boundaries, constraint violations, race conditions.
4. **Performance** (P3): N+1 queries, missing indexes, memory leaks, blocking I/O.
5. **Maintainability** (P4): SOLID/DRY violations, high coupling, low cohesion.
6. **Style** (P5): Naming conventions, formatting, comment quality.

Rule: DO NOT spend review time on P5 issues while P0-P2 issues remain unaddressed.

## 2. Severity Classification Guide

### Critical
- Causes data loss, corruption, or security breach.
- Results in system crash or unrecoverable state.
- Violates regulatory or compliance requirements.
- Examples: SQL injection, missing auth check on admin endpoint, unbounded query in production.

### High
- Causes incorrect business behavior under normal conditions.
- Breaks API contract or DB schema agreement.
- Missing critical-path test coverage.
- Examples: Wrong calculation logic, silent exception swallowing, blueprint violation.

### Medium
- Causes incorrect behavior under edge/error conditions.
- Creates technical debt that increases future failure risk.
- Performance degradation under expected load.
- Examples: Missing null check, N+1 query, duplicated business logic.

### Low
- Style and readability improvements.
- Minor naming inconsistency.
- Non-critical code structure suggestion.
- Examples: Variable naming, comment wording, import ordering.

## 3. Security Review Checklist
For every code review, inspect:

### Injection Prevention
- [ ] SQL queries use parameterized statements (no string concatenation).
- [ ] User input is validated and sanitized before processing.
- [ ] Dynamic queries (if any) use safe query builders.
- [ ] Log messages do not include unsanitized user input.

### Authentication and Authorization
- [ ] All endpoints have appropriate auth guards.
- [ ] Role-based access controls match blueprint specifications.
- [ ] Token validation is present and correct.
- [ ] Session management follows security best practices.

### Secrets and Configuration
- [ ] No hardcoded passwords, API keys, or tokens in source code.
- [ ] Secrets are loaded from environment variables or secret management.
- [ ] Configuration files do not contain production credentials.
- [ ] `.gitignore` properly excludes sensitive files.

### Data Protection
- [ ] Sensitive data is not exposed in API responses.
- [ ] Error messages do not leak internal structure or stack traces.
- [ ] PII is handled according to data protection requirements.
- [ ] Audit logging captures security-relevant actions.

### CSRF/CORS
- [ ] CSRF protection is enabled for state-changing operations.
- [ ] CORS configuration restricts origins appropriately.

## 4. Blueprint Compliance Check Protocol
Before approving code, verify against Tech Lead blueprint:

### Hard Gate Checks (Blocking)
| Check Point | How To Verify |
|---|---|
| API Response Schema | Compare response payload structure with OpenAPI definition |
| API Error Model | Verify error codes and messages match blueprint error contract |
| DB Entity Structure | Compare JPA entity with blueprint ERD (fields, types, constraints) |
| Transaction Boundaries | Verify @Transactional placement matches blueprint transaction rules |
| Security Implementation | Verify auth/authz matches blueprint security specification |

### Soft Gate Checks (Warning)
| Check Point | How To Verify |
|---|---|
| Layer Boundaries | Controller → Service → Repository separation maintained |
| Module Placement | New classes are in the correct package/module per architecture |
| Naming Convention | Class/method/variable naming follows project standards |
| Error Handling Pattern | Exception hierarchy follows blueprint error strategy |
| Logging Standards | Structured logging with correlation IDs per blueprint |

### Compliance Reporting Format
For each blueprint deviation found:
```
- Deviation ID: BP-DEV-<number>
- Gate Type: Hard | Soft
- Blueprint Reference: <section or document>
- Code Reference: <file:line>
- Deviation: <what does not match>
- Required Change: <how to fix>
```

## 5. Test Coverage Requirements

### By Risk Level
| Risk Level | Unit Test | Integration Test | E2E Test |
|---|---|---|---|
| Critical Path | ✅ Required (positive + negative) | ✅ Required | ✅ Recommended |
| High Risk | ✅ Required (positive + negative) | ✅ Required | ⚪ Optional |
| Medium Risk | ✅ Required (positive) | ⚪ Optional | ⚪ Optional |
| Low Risk | ⚪ Recommended | ⚪ Optional | ⚪ Optional |

### Test Quality Standards
- [ ] Tests have descriptive names that explain the scenario and expected result.
- [ ] Tests are independent and do not depend on execution order.
- [ ] Tests use proper setup/teardown and do not leak state.
- [ ] Assert statements verify behavior, not implementation details.
- [ ] Edge cases are covered: null, empty, boundary values, error conditions.
- [ ] Test data is realistic and representative.

### Coverage Gate
- High-risk change with no critical-path tests → **Blocked**.
- Medium-risk change with no unit tests → **Warning** with follow-up task.
- Low-risk change with no tests → **Accepted** with recommendation note.

## 6. Merge Gate Decision Tree

```
START: Review Complete
│
├─ Any Critical finding unresolved?
│   ├─ YES → BLOCKED (must fix before merge)
│   └─ NO ↓
│
├─ Any High finding unresolved?
│   ├─ YES → Is it in a critical user path?
│   │   ├─ YES → BLOCKED
│   │   └─ NO → Can it be fixed in follow-up?
│   │       ├─ YES → CONDITIONAL (merge with tracked follow-up)
│   │       └─ NO → BLOCKED
│   └─ NO ↓
│
├─ Any Hard Gate blueprint deviation?
│   ├─ YES → BLOCKED
│   └─ NO ↓
│
├─ Critical-path tests missing?
│   ├─ YES → BLOCKED
│   └─ NO ↓
│
├─ Only Medium/Low findings remaining?
│   └─ YES → APPROVED (with improvement notes)
│
└─ No findings at all?
    └─ APPROVED
```

### Merge Gate Output
```
- Gate Status: Approved | Conditional | Blocked
- Blocking Findings: <finding IDs>
- Conditions for Conditional: <what must be done post-merge>
- Follow-up Tracker: <task IDs or descriptions>
- Reviewer Sign-off: @code-reviewer
```

## 7. Rework Request Protocol
When requesting fixes from developer agents:

### Rework Request Template
```
## Rework Request: RV-<finding_id>

### What Is Wrong
<Specific description with file:line reference>

### Why It Matters
<Business/security/performance impact>

### Evidence
<Code snippet, test failure, or reproduction steps>

### Suggested Fix Direction
<How to approach the fix — not necessarily exact code>

### Acceptance Criteria for Fix
- <Criterion 1>
- <Criterion 2>

### Priority
Critical | High | Medium | Low

### Deadline Expectation
<Before next review cycle | Before sprint end | Follow-up>
```

### Rework Rules
- Every rework request must include evidence and fix direction.
- Critical/High rework must be addressed before next review round.
- Medium/Low rework can be batched into follow-up PR if approved by reviewer.
- Developer must reference the finding ID when submitting fix.

## 8. Review Workflow Steps

1. **Context Load**
   - Read PR description, linked user stories, acceptance criteria.
   - Read relevant blueprint sections (API contract, DB schema, architecture).
   - Check `/PROJECT_STATE.md` for current stage and constraints.

2. **Diff Analysis**
   - Review changed files for correctness and logic.
   - Check neighboring code for side effects and dependency impact.
   - Identify changes that affect public interfaces or contracts.

3. **Security Scan**
   - Run through security checklist (Section 3).
   - Invoke `/security-vulnerability-scanning` skill if high-risk changes detected.

4. **Blueprint Compliance**
   - Run hard gate and soft gate checks (Section 4).
   - Document all deviations.

5. **Test Validation**
   - Verify test coverage against risk level requirements (Section 5).
   - Invoke `/automated-quality-gate` skill for AC coverage analysis.

6. **Clean Code Check**
   - Apply SOLID/DRY inspection.
   - Invoke `/clean-code-patterns-enforcement` skill for refactoring suggestions.

7. **Findings Compilation**
   - Classify all findings by severity.
   - Generate blocking decision using merge gate decision tree (Section 6).

8. **Output Delivery**
   - Return structured review output per agent output format.
   - Issue rework requests for blocking findings.
   - Update `/PROJECT_STATE.md` with review status.

## 9. Output Quality Gate (Mandatory)
Review task is NOT complete unless:

- [ ] All changed files have been inspected.
- [ ] Security checklist has been executed.
- [ ] Blueprint compliance checks (hard gate minimum) have been performed.
- [ ] Test coverage has been validated against risk level.
- [ ] Findings are classified with severity, evidence, and fix direction.
- [ ] Merge gate decision is explicitly stated with rationale.
- [ ] Rework requests (if any) are issued with complete template.
- [ ] `/PROJECT_STATE.md` updated with review outcome.

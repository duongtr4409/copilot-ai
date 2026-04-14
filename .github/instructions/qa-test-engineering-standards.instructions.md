---
description: "Use when QA designs test strategies, creates test plans from AC, writes test cases, executes test scenarios, manages test data, reports defects, and makes quality gate decisions for release readiness."
name: "QA Test Engineering Standards"
---

# QA Test Engineering Standards Instruction

## Table of Contents
1. Test Strategy Per Environment
2. Test Case Writing Standards
3. Test Data Management Protocol
4. Automation Framework Conventions
5. Regression Test Suite Maintenance
6. Bug Report Quality Standards
7. Quality Gate Threshold Definitions
8. Release Readiness Checklist
9. Performance Test Baseline Protocol
10. Output Quality Gate

## 1. Test Strategy Per Environment

### Development (Dev)
| Test Type | Scope | Responsibility | Automation |
|---|---|---|---|
| Unit Tests | Individual functions/methods | Developer | Must be automated |
| Component Tests | Service/module boundaries | Developer + QA | Automated preferred |
| Smoke Tests | Critical path sanity | QA | Automated |

### Staging
| Test Type | Scope | Responsibility | Automation |
|---|---|---|---|
| Integration Tests | Cross-module, API contracts | QA + Developer | Must be automated |
| E2E Tests | Full user journeys | QA | Automated preferred |
| Regression Suite | All existing features | QA | Must be automated |
| Security Tests | OWASP-based checks | QA + Security Auditor | Semi-automated |
| Performance Baseline | Response time, throughput | QA + DevOps | Automated |

### Production
| Test Type | Scope | Responsibility | Automation |
|---|---|---|---|
| Smoke Tests | Critical user paths | QA + DevOps | Automated |
| Health Checks | System availability | DevOps | Automated (monitoring) |
| Canary Validation | New version behavior | DevOps + QA | Automated |

## 2. Test Case Writing Standards

### Derivation from AC
Every test case MUST trace to one or more Acceptance Criteria:

```
Test Case ID: TC-<feature>-<number>
Linked AC: AC-<number>
Priority: P0 | P1 | P2 | P3
Type: Functional | Integration | E2E | Security | Performance
Preconditions: <setup requirements>
```

### Gherkin Format (Mandatory for Functional Tests)
```gherkin
Feature: User Registration

  Scenario: Successful registration with valid data
    Given the user is on the registration page
    And no account exists with email "test@example.com"
    When the user submits the form with:
      | Field    | Value              |
      | name     | Test User          |
      | email    | test@example.com   |
      | password | SecurePass123!     |
    Then the system creates a new user account
    And the user receives a confirmation email
    And the response status is 201

  Scenario: Registration fails with duplicate email
    Given an account already exists with email "test@example.com"
    When the user submits the form with email "test@example.com"
    Then the response status is 409
    And the error message is "Email already registered"
```

### Test Case Categories (Must Cover All)
| Category | Description | Examples |
|---|---|---|
| Happy Path | Normal successful flow | Create, read, update, delete successfully |
| Negative Path | Invalid input, forbidden actions | Invalid email, unauthorized access, duplicate submit |
| Boundary Values | Min/max/edge limits | Empty string, max length, zero, negative numbers |
| Error Handling | System failures, timeouts | DB down, external service timeout, network error |
| State Transitions | Entity lifecycle | Draft → Published → Archived → Deleted |
| Concurrency | Parallel operations | Double submit, race condition on inventory |
| Security | Auth/authz edge cases | Expired token, role escalation, CSRF |

## 3. Test Data Management Protocol

### Principles
- Test data MUST be deterministic and reproducible.
- Tests MUST NOT depend on production data.
- Each test should create its own data or use shared fixtures with known state.
- Tests MUST clean up after themselves (or use transactional rollback).

### Test Data Layers
| Layer | Purpose | Tools |
|---|---|---|
| Fixtures | Static reference data (roles, statuses, config) | SQL scripts, seed files |
| Factories | Dynamic test entity generation | Factory pattern, Faker libraries |
| Snapshots | Point-in-time DB state for complex scenarios | DB dump/restore, Testcontainers |

### Sensitive Data Rules
- NEVER use real production data in test environments.
- Use anonymized or synthetic data that preserves data shape.
- PII in test data must use obviously fake values (e.g., `test_user_001@test.local`).

### Test Data Naming Convention
- Test user emails: `test_[scenario]@test.local`
- Test IDs: Use predictable patterns (e.g., `TEST-USER-001`)
- Test phone numbers: Use invalid prefixes (e.g., `+0000000001`)

## 4. Automation Framework Conventions

### Backend Test Stack
| Tool | Purpose |
|---|---|
| JUnit 5 | Test runner and assertions |
| Mockito | Mocking dependencies |
| AssertJ | Fluent assertions |
| Testcontainers | Database integration testing |
| REST Assured | API integration testing |
| WireMock | External service mocking |

### Frontend Test Stack
| Tool | Purpose |
|---|---|
| Jest | Unit test runner (React) |
| Jasmine/Karma | Unit test runner (Angular) |
| Cypress or Playwright | E2E browser testing |
| Testing Library | Component testing |
| MSW (Mock Service Worker) | API mocking |

### Automation Rules
- Automated tests MUST be runnable via `mvn test` or `npm test` without manual setup.
- Automated tests MUST pass in CI/CD pipeline.
- Flaky tests MUST be identified, quarantined, and fixed within 1 sprint.
- Test execution time: unit tests < 5 min, integration tests < 15 min, E2E < 30 min.

### Test Report Format
- Generate HTML/XML report per test run.
- Include: total tests, passed, failed, skipped, duration, failure details.
- Attach report to PR or release evidence.

## 5. Regression Test Suite Maintenance

### Suite Organization
```
tests/
├── smoke/              # Critical path (run on every PR)
├── regression/         # Full feature coverage (run on staging)
│   ├── auth/
│   ├── user/
│   ├── order/
│   └── payment/
├── integration/        # Cross-module tests
├── e2e/               # End-to-end user journeys
└── performance/       # Load and stress tests
```

### Maintenance Rules
- When a new feature is released, add regression tests within the same sprint.
- When a bug is fixed, add a regression test that covers the fixed scenario.
- Review regression suite quarterly for:
  - Redundant/overlapping tests → consolidate.
  - Outdated tests for removed features → archive.
  - Missing coverage for new features → add.
- Tag tests by feature area and priority for selective execution.

### Smoke Test Criteria
Smoke suite MUST cover:
- [ ] User authentication (login/logout)
- [ ] Core CRUD operations for primary entities
- [ ] Critical business workflow (e.g., checkout, payment)
- [ ] API health endpoints
- [ ] Database connectivity

## 6. Bug Report Quality Standards

### Mandatory Fields
```
Bug ID: BUG-<number>
Title: <concise summary of the defect>
Severity: Critical | High | Medium | Low
Priority: P0 | P1 | P2 | P3
Linked AC: AC-<number>
Environment: Dev | Staging | Production
Reporter: @qa-tester

## Reproduction Steps
1. <Step 1>
2. <Step 2>
3. <Step 3>

## Expected Result
<What should have happened>

## Actual Result
<What actually happened>

## Evidence
- Screenshot/Video: <attachment>
- Logs: <relevant log excerpt>
- Test Output: <test failure trace>

## Suggested Fix Direction
<High-level guidance on possible root cause and approach>

## Impact Assessment
- Users Affected: <scope>
- Data Impact: <any data corruption or loss>
- Workaround Available: Yes/No — <description if yes>
```

### Bug Severity Definitions
| Severity | Definition | SLA |
|---|---|---|
| Critical | System down, data loss, security breach | Fix within 4 hours |
| High | Major feature broken, no workaround | Fix within 1 business day |
| Medium | Feature partially broken, workaround exists | Fix within current sprint |
| Low | Cosmetic, minor UX issue | Fix in backlog |

### Bug Report Rules
- EVERY bug must have reproduction steps.
- EVERY bug must link to the relevant AC or user story.
- NEVER mark a bug as "Cannot Reproduce" without at least 3 reproduction attempts.
- Include environment details (browser, OS, API version) for frontend bugs.
- Include request/response payloads for API bugs.

## 7. Quality Gate Threshold Definitions

### Gate Levels
```
PASS:
  - Zero Critical defects.
  - Zero High defects in critical user paths.
  - All Critical AC fully tested and passing.
  - Test coverage meets minimum threshold.

WARNING (Conditional Pass):
  - Zero Critical defects.
  - High defects exist but NOT in critical user paths.
  - Mitigation or workaround documented.
  - Follow-up fix committed to next sprint.

BLOCKED (Fail):
  - Any Critical defect unresolved.
  - Any High defect in critical user path.
  - Any Critical AC untested or failing.
  - Missing security test evidence for auth-sensitive features.
```

### Coverage Thresholds
| Metric | Minimum | Target |
|---|---|---|
| AC covered by test cases | 100% | 100% |
| Critical path test automation | 80% | 95% |
| Unit test code coverage | 70% | 85% |
| Integration test coverage | 50% | 70% |

## 8. Release Readiness Checklist
QA MUST verify all items before recommending release:

### Functional Readiness
- [ ] All Critical and High AC test cases passed.
- [ ] All regression tests passed on staging.
- [ ] Smoke tests passed on release candidate.
- [ ] No Critical or High defects unresolved.
- [ ] All bug fixes have regression tests.

### Non-Functional Readiness
- [ ] Performance baseline met (response time, throughput).
- [ ] Security scan completed (no critical findings).
- [ ] Accessibility compliance verified (if applicable).

### Documentation Readiness
- [ ] Test execution report generated and archived.
- [ ] Defect list with resolution status documented.
- [ ] Known issues documented with workarounds.
- [ ] Release notes reviewed for accuracy.

### Sign-off
```
Release Candidate: <version>
QA Decision: PASS | WARNING | BLOCKED
Tested By: @qa-tester
Test Report: <link>
Defect Summary: Critical: 0, High: 0, Medium: X, Low: Y
Known Issues: <list or none>
Date: YYYY-MM-DDTHH:MM:SS+07:00
```

## 9. Performance Test Baseline Protocol

### When to Run
- Before first release of a new service/feature.
- After significant architecture or DB schema changes.
- Before major release milestones.
- Quarterly for established services.

### Metrics to Capture
| Metric | Tool | Threshold |
|---|---|---|
| P50/P95/P99 Response Time | JMeter, k6, Gatling | Per API endpoint |
| Throughput (RPS) | JMeter, k6 | Per API endpoint |
| Error Rate | Test tool + monitoring | < 0.1% under normal load |
| DB Query Time | EXPLAIN/ANALYZE, APM | P95 < 100ms |
| Memory Usage | JVM metrics, APM | No memory leak pattern |
| CPU Usage | System metrics | < 70% under normal load |

### Test Scenarios
| Scenario | Description | Duration |
|---|---|---|
| Baseline | Normal expected load | 10 min |
| Peak | 2x normal load | 5 min |
| Stress | Increasing load until failure | Until error threshold |
| Soak | Sustained normal load | 1 hour |

### Performance Report Format
```
## Performance Test Report
- Date: YYYY-MM-DD
- Environment: Staging
- Test Tool: <JMeter/k6/Gatling>
- Build Version: <version>

### Results Summary
| Endpoint | Method | P50 | P95 | P99 | RPS | Error% | Status |
|---|---|---|---|---|---|---|---|

### Comparison with Baseline
| Metric | Baseline | Current | Delta | Status |
|---|---|---|---|---|

### Findings and Recommendations
- <findings>

### Verdict: PASS | DEGRADED | FAIL
```

## 10. Output Quality Gate (Mandatory)
QA task is NOT complete unless:

- [ ] Test strategy defined per environment.
- [ ] All Critical/High AC have test cases with Gherkin format.
- [ ] Test data is deterministic and documented.
- [ ] Test execution completed with all scenarios.
- [ ] Bug reports meet quality standards (reproduction steps, evidence, AC link).
- [ ] Quality gate decision explicitly stated with rationale.
- [ ] Release readiness checklist completed (if release scope).
- [ ] Test execution report generated.
- [ ] `/PROJECT_STATE.md` updated with QA status and findings.

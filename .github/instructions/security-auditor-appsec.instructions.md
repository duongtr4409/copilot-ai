---
description: "Use when Security Auditor performs application security assessments: OWASP Top 10 mapping, SAST/DAST workflows, dependency CVE auditing, secrets detection, compliance checks, and pre-approval security gate decisions."
name: "Security Auditor AppSec Standards"
---

# Security Auditor AppSec Standards Instruction

## Table of Contents
1. OWASP Top 10 Mapping and Checklist
2. SAST Workflow
3. Dependency Vulnerability Audit
4. Secrets Detection Protocol
5. Security Gate Decision Matrix
6. Compliance and Regulatory Checks
7. Remediation Priority Framework
8. Security Exception Management
9. Output Quality Gate

## 1. OWASP Top 10 Mapping and Checklist
Every security audit MUST map findings to OWASP Top 10 (2021):

| # | OWASP Category | Check Focus | Priority |
|---|---|---|---|
| A01 | Broken Access Control | Role checks, IDOR, path traversal, CORS | Critical |
| A02 | Cryptographic Failures | TLS config, password hashing, data encryption at rest | Critical |
| A03 | Injection | SQLi, XSS, command injection, LDAP injection, SSRF | Critical |
| A04 | Insecure Design | Threat modeling, business logic flaws, missing rate limits | High |
| A05 | Security Misconfiguration | Default credentials, unnecessary features, error disclosure | High |
| A06 | Vulnerable Components | Known CVEs in dependencies, outdated libraries | High |
| A07 | Auth Failures | Brute force, credential stuffing, session management | Critical |
| A08 | Data Integrity Failures | Unsigned updates, insecure deserialization, CI/CD integrity | High |
| A09 | Logging/Monitoring Failures | Missing audit logs, no alerting, insufficient log detail | Medium |
| A10 | SSRF | Server-side request forgery via user-controlled URLs | High |

### Per-Review Checklist
For each code review or audit:
- [ ] **A01**: All endpoints verify user authorization (not just authentication).
- [ ] **A01**: No direct object reference without ownership validation.
- [ ] **A02**: Passwords hashed with bcrypt/argon2 (not MD5/SHA1).
- [ ] **A02**: Sensitive data encrypted in transit (TLS 1.2+) and at rest.
- [ ] **A03**: All SQL queries fully parameterized.
- [ ] **A03**: User input HTML-escaped before rendering (XSS prevention).
- [ ] **A03**: No OS command construction from user input.
- [ ] **A04**: Rate limiting on authentication and public endpoints.
- [ ] **A05**: No default or hardcoded credentials in config.
- [ ] **A05**: Error responses do not leak stack traces or internal paths.
- [ ] **A06**: No Critical/High CVE in production dependencies.
- [ ] **A07**: Session tokens regenerated after authentication.
- [ ] **A07**: JWT tokens have appropriate expiry and validation.
- [ ] **A08**: API input validated and deserialized safely.
- [ ] **A09**: Security events logged (login, failed auth, role changes, data access).
- [ ] **A10**: External URL requests validated against allowlist.

## 2. SAST Workflow

### When to Run
- On every PR that touches: controllers, services, security config, DB queries, auth logic.
- Full scan before release branch creation.
- Scheduled weekly scan on main branch.

### SAST Focus Areas
| Category | Detection Target | Tool Guidance |
|---|---|---|
| Injection | SQL, XSS, Command injection, SSRF | Semgrep, SonarQube, CodeQL |
| Secrets | API keys, tokens, passwords in code | Gitleaks, TruffleHog, detect-secrets |
| Auth | Missing auth checks, insecure session handling | Custom rules + manual review |
| Crypto | Weak algorithms, insecure random | Semgrep rules, manual review |
| Config | Unsafe defaults, debug mode, verbose errors | Config file review |

### SAST Process
1. Define scan scope (changed files or full repo).
2. Run SAST tool with project-specific rule set.
3. Review findings — classify as True Positive or False Positive.
4. For True Positive: create security finding with severity and fix direction.
5. For False Positive: document rationale and add suppression with expiry date.
6. Re-scan after remediation to confirm fix.

## 3. Dependency Vulnerability Audit

### Scan Targets
| Ecosystem | Manifest Files | Tool Options |
|---|---|---|
| Java/Maven | `pom.xml` | OWASP Dependency-Check, Snyk |
| Java/Gradle | `build.gradle` | OWASP Dependency-Check, Snyk |
| Node.js | `package.json`, `package-lock.json` | npm audit, Snyk, Socket |
| Python | `requirements.txt`, `Pipfile.lock` | pip-audit, Safety, Snyk |

### Severity Response Matrix
| CVE Severity | Production Dep | Dev/Test Dep | Action |
|---|---|---|---|
| Critical | 🔴 Block merge | ⚠️ Warning | Immediate upgrade/patch |
| High | 🔴 Block merge | ⚠️ Warning | Upgrade within sprint |
| Medium | ⚠️ Warning | 📝 Note | Track and plan upgrade |
| Low | 📝 Note | 📝 Note | Backlog |

### Audit Report Format
```
Dependency: <package@version>
CVE ID: CVE-YYYY-XXXXX
Severity: Critical | High | Medium | Low
CVSS Score: <score>
Affected Version: <range>
Fixed Version: <version>
Exploitability: Proven | Theoretical | Unknown
Production Reachable: Yes | No | Unknown
Remediation: Upgrade to <version> | Replace with <alternative> | Accept risk
Timeline: Immediate | This sprint | Next sprint | Backlog
```

## 4. Secrets Detection Protocol

### Detection Scope
- Source code files (all languages).
- Configuration files (`.yml`, `.yaml`, `.json`, `.env`, `.properties`).
- Infrastructure files (Terraform, Dockerfile, CI/CD configs).
- Git history (historical commits may contain rotated secrets).

### Secret Types to Detect
| Type | Pattern Examples |
|---|---|
| API Keys | `AKIA...`, `sk-...`, `ghp_...` |
| Database Credentials | `password=...`, `jdbc:...` with embedded credentials |
| JWT/Session Secrets | `JWT_SECRET=`, signing keys |
| Private Keys | `-----BEGIN RSA PRIVATE KEY-----` |
| OAuth Tokens | `ya29.`, `xoxb-`, `Bearer <token>` |
| Cloud Credentials | AWS access keys, Azure connection strings, GCP service account JSON |

### Response Protocol
When a secret is detected:
1. **Severity**: Treat as Critical incident.
2. **Immediate Actions**:
   - Revoke/rotate the exposed credential.
   - Remove from source code.
   - Move to secret management (env vars, Vault, cloud secret manager).
   - Check git history — if secret was committed, consider force-push or repo rotation.
3. **Post-Incident**:
   - Add pattern to `.gitignore` or pre-commit hook.
   - Add detection rule to CI pipeline.
   - Document incident in security log.

## 5. Security Gate Decision Matrix

```
START: Security Audit Complete
│
├─ Any Critical OWASP finding (A01-A03, A07)?
│   ├─ YES → BLOCKED (must fix before merge/release)
│   └─ NO ↓
│
├─ Any Critical CVE in production dependency?
│   ├─ YES → BLOCKED
│   └─ NO ↓
│
├─ Any active secret in source code?
│   ├─ YES → BLOCKED (incident response)
│   └─ NO ↓
│
├─ Any High OWASP finding?
│   ├─ YES → Is it in critical user path?
│   │   ├─ YES → BLOCKED
│   │   └─ NO → WARNING (fix within sprint, document risk)
│   └─ NO ↓
│
├─ Any High CVE in production dependency?
│   ├─ YES → WARNING (fix within sprint, document risk)
│   └─ NO ↓
│
├─ Only Medium/Low findings?
│   └─ PASS (with improvement notes and tracking)
│
└─ No findings
    └─ PASS
```

## 6. Compliance and Regulatory Checks
When applicable, verify:

| Regulation | Key Requirements | Check Method |
|---|---|---|
| GDPR | Data minimization, consent, right to erasure, breach notification | Code + architecture review |
| PCI DSS | Cardholder data protection, access control, audit trail | Code + infra review |
| HIPAA | PHI encryption, access controls, audit logging | Code + infra review |
| SOC 2 | Security, availability, processing integrity, confidentiality | Process + code review |

## 7. Remediation Priority Framework

| Priority | Criteria | SLA |
|---|---|---|
| P0 - Emergency | Active exploit, data breach, secret exposure | 4 hours |
| P1 - Critical | Exploitable vulnerability in production path | 1 business day |
| P2 - High | Exploitable vulnerability in non-critical path | Current sprint |
| P3 - Medium | Theoretical vulnerability, defense-in-depth gap | Next sprint |
| P4 - Low | Best practice improvement, hardening suggestion | Backlog |

## 8. Security Exception Management

### Exception Request Template
```
Exception ID: SEC-EXP-<number>
Finding Reference: SEC-<number>
Requested By: @<agent>
Approved By: @security-auditor + @tech-lead
Reason: <why exception is needed>
Risk Assessment: <residual risk with exception>
Compensating Controls: <what mitigates the risk>
Expiry Date: YYYY-MM-DD
Review Cadence: <monthly | quarterly>
Remediation Plan: <what will ultimately fix this>
```

### Exception Rules
- Critical findings cannot have exceptions (must be fixed).
- High findings require Tech Lead + PMO approval for exception.
- All exceptions must have an expiry date (max 90 days).
- Expired exceptions automatically escalate to BLOCKED status.

## 9. Output Quality Gate (Mandatory)
Security audit task is NOT complete unless:

- [ ] OWASP Top 10 checklist executed for relevant categories.
- [ ] SAST scan completed for changed code scope.
- [ ] Dependency vulnerability audit completed.
- [ ] Secrets detection scan completed.
- [ ] All findings classified with severity, evidence, and fix direction.
- [ ] Security gate decision explicitly stated.
- [ ] Remediation plan provided for all blocking findings.
- [ ] Exceptions documented with approval and expiry dates.
- [ ] `/PROJECT_STATE.md` updated with security audit status.

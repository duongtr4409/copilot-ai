---
name: git-workflow-enforcement
description: "Enforce Git branching strategy, commit message conventions, PR templates, branch protection rules, and merge policies. Use for repository setup, PR quality gates, and Git flow compliance across the team."
argument-hint: "Repository context, branching strategy, commit convention, PR requirements, and team size"
user-invocable: true
---

# Git Workflow Enforcement (Chuan hoa quy trinh Git)

## What This Skill Produces
- Validated branch naming and merge rules before code changes.
- Conventional commit messages that enable automated changelog generation.
- PR template compliance for consistent review context.
- Branch protection configuration aligned with delivery pipeline.
- Git hygiene: clean history, no force-push to protected branches, proper tagging.

## When To Use
- Setting up a new repository for the agent team.
- Reviewing PRs for Git workflow compliance.
- Onboarding new agents or developers to repository conventions.
- Debugging merge conflicts or branch management issues.
- Configuring branch protection rules.

## Required Inputs
- Repository URL and platform (GitHub/GitLab).
- Team branching strategy (Git Flow, GitHub Flow, Trunk-based).
- Release cadence (sprint-based, continuous, manual).
- Team size and approval requirements.

## Branching Strategy

### Default: Feature Branch Flow
```
main                    ← production-ready, protected
├── develop             ← integration branch (optional, for Git Flow)
├── feature/US-001-login ← feature work
├── bugfix/BUG-042-null  ← bug fixes
├── hotfix/SEC-001-xss   ← urgent production fixes
└── release/v1.2.0       ← release preparation (optional)
```

### Branch Naming Convention
| Type | Pattern | Example |
|---|---|---|
| Feature | `feature/<story-id>-<short-desc>` | `feature/US-001-user-login` |
| Bug Fix | `bugfix/<bug-id>-<short-desc>` | `bugfix/BUG-042-null-pointer` |
| Hotfix | `hotfix/<finding-id>-<short-desc>` | `hotfix/SEC-001-xss-fix` |
| Release | `release/v<major.minor.patch>` | `release/v1.2.0` |
| Spike/Research | `spike/<topic>` | `spike/redis-caching` |

### Branch Rules
- Direct commits to `main` are PROHIBITED.
- All changes must go through a PR (Pull Request).
- Feature branches must be created from `main` (or `develop` if using Git Flow).
- Feature branches must be deleted after merge.
- Branch name must include ticket reference (US/BUG/SEC ID).

## Commit Message Convention

### Format: Conventional Commits
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
| Type | When to Use | Example |
|---|---|---|
| `feat` | New feature | `feat(auth): add JWT token refresh endpoint` |
| `fix` | Bug fix | `fix(order): correct total calculation for discounts` |
| `refactor` | Code restructuring, no behavior change | `refactor(user): extract validation to service layer` |
| `test` | Adding or fixing tests | `test(payment): add edge case for zero amount` |
| `docs` | Documentation only | `docs(api): update OpenAPI schema for order endpoints` |
| `chore` | Build, CI, config changes | `chore(ci): add security scan stage to pipeline` |
| `perf` | Performance improvement | `perf(query): add composite index for order search` |
| `security` | Security fix | `security(auth): fix session fixation vulnerability` |
| `style` | Formatting, no logic change | `style(user): fix indentation in UserService` |

### Commit Message Rules
- Subject line max 72 characters.
- Subject uses imperative mood ("add" not "added" or "adds").
- Body explains WHY, not WHAT (the diff shows what changed).
- Footer references ticket: `Refs: US-001` or `Closes: BUG-042`.
- Breaking changes: add `BREAKING CHANGE:` in footer.

### Bad vs Good Examples
```
# BAD
fix bug
update code
WIP
misc changes

# GOOD
fix(auth): prevent session fixation on login
feat(order): add bulk order creation endpoint
refactor(payment): separate payment gateway adapter
```

## Pull Request Template

### PR Template (`/.github/pull_request_template.md`)
```markdown
## Summary
<!-- Brief description of what this PR does and why -->

## Linked Issues
<!-- Reference tickets: Closes #123, Refs US-001 -->

## Type of Change
- [ ] Feature (new functionality)
- [ ] Bug Fix (fixes an issue)
- [ ] Refactor (no behavior change)
- [ ] Documentation
- [ ] CI/CD / Infrastructure
- [ ] Security Fix

## Changes Made
<!-- List key changes with file references -->

## Testing Done
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed
- [ ] Test evidence: <!-- link or description -->

## Blueprint Compliance
- [ ] API contract matches OpenAPI spec
- [ ] DB schema matches blueprint
- [ ] Security controls match spec
- [ ] N/A (not applicable)

## Checklist
- [ ] Code follows project conventions
- [ ] No hardcoded secrets or credentials
- [ ] Error handling is appropriate
- [ ] Logging follows standards
- [ ] Documentation updated if needed
- [ ] No unnecessary console.log or debug statements
```

## Branch Protection Rules

### Main Branch
| Rule | Value |
|---|---|
| Require PR before merge | ✅ Yes |
| Required approving reviews | 1 (min), 2 (recommended) |
| Dismiss stale reviews on new push | ✅ Yes |
| Require status checks to pass | ✅ Yes (CI pipeline) |
| Require conversation resolution | ✅ Yes |
| Require linear history | ✅ Yes (squash merge) |
| Allow force push | ❌ No |
| Allow deletion | ❌ No |

### Merge Strategy
- Default: **Squash and merge** (clean linear history).
- Release branches: **Merge commit** (preserve history).
- NEVER use rebase on shared branches.

## Decision Logic And Branching

1. If commit message does not follow Conventional Commits:
   - Reject PR until commit message is fixed.
   - Suggest correct format with example.

2. If branch name does not follow naming convention:
   - Warn and request branch rename before merge.

3. If PR template sections are incomplete:
   - Request PR author to fill all required sections.
   - "Testing Done" and "Blueprint Compliance" are mandatory for feature/bugfix PRs.

4. If PR has merge conflicts:
   - Author must resolve conflicts before review starts.
   - Reviewer should NOT resolve conflicts on behalf of author.

5. If direct commit to `main` is detected:
   - Flag as policy violation.
   - Require reversal via PR if possible.
   - Report in daily report.

## Workflow Integration
- PMO: Validate branch creation follows naming convention when assigning tasks.
- Developers: Create branch with correct naming before starting work.
- Code Reviewer: Verify PR template completeness and commit convention in review.
- Technical Writer: Use commit history (Conventional Commits) for changelog generation.

## Output Contract
When this skill is invoked, return:

1. Git Workflow Assessment
2. Branch Naming Compliance
3. Commit Convention Compliance
4. PR Template Compliance
5. Branch Protection Recommendations
6. Violations and Remediation Actions

## Example Invocations
- /git-workflow-enforcement Setup branch protection and PR template for a new repository.
- /git-workflow-enforcement Review this PR for Git workflow and commit convention compliance.
- /git-workflow-enforcement Generate Conventional Commits changelog from recent merge history.

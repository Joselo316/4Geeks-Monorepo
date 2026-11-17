# HealthCore Digital — Obligatory Commit Workflow

**Version:** 1.0  
**Last Updated:** November 17, 2026  
**Applies to:** All commits to `main`, `develop`, and feature branches  
**Owner:** James Osei (CTO)  
**Enforcement:** GitHub branch protection rules + CI/CD pipeline

---

## 🎯 Purpose

This document defines the **mandatory workflow that every developer must follow before committing code** to the HealthCore Digital monorepo. The workflow ensures:

✅ Code quality  
✅ Compliance with HIPAA/GDPR  
✅ Test coverage  
✅ Performance standards  
✅ Documentation completeness  
✅ Consistency across the team

**This is non-negotiable.** No commit bypasses this workflow.

---

## 📋 Minimum 4-Step Obligatory Workflow

### **STEP 1: Code Quality & Type Safety** (5-10 min)

**Purpose:** Ensure code follows standards and has no syntax errors.

#### Sub-steps:

1. **Format code with Black (Python)**
   ```bash
   black services/api --line-length 100
   ```
   - Enforces consistent formatting across team
   - Non-negotiable: "We don't discuss tabs vs spaces"

2. **Lint code with Ruff (Python)**
   ```bash
   ruff check services/api --fix
   ```
   - Catches unused imports, undefined variables, security issues
   - `--fix` automatically corrects common issues

3. **Format code with Prettier (TypeScript/JavaScript)**
   ```bash
   prettier uis/backoffice/src --write
   ```
   - Enforces consistent formatting for frontend code

4. **Lint code with ESLint (TypeScript/JavaScript)**
   ```bash
   eslint uis/backoffice/src --fix
   ```
   - Catches React best practices, unused code, security issues

5. **Type check with Mypy (Python)**
   ```bash
   mypy services/api
   ```
   - Verifies all type annotations are correct
   - Catches type mismatches before tests run

6. **Type check with TypeScript compiler**
   ```bash
   tsc --noEmit uis/backoffice
   ```
   - Verifies all TypeScript is type-safe

**Checklist:**
- [ ] `black` passed without errors
- [ ] `ruff` passed without errors
- [ ] `prettier` applied (no changes rejected)
- [ ] `eslint` passed without errors
- [ ] `mypy` passed without errors
- [ ] `tsc` passed without errors (zero type errors)

**If Any Check Fails:**
1. Review the error message carefully
2. Fix the code (or suppress explicitly with comments if justified)
3. Re-run the check
4. If still failing, escalate to Tech Lead (James Osei) with error log

**Definition of Done for Step 1:**
```
✅ All formatting tools pass
✅ All linting tools pass
✅ All type checking passes
✅ No warnings in any tool
```

---

### **STEP 2: Automated Tests** (10-20 min)

**Purpose:** Ensure new code doesn't break existing functionality and has adequate coverage.

#### Sub-steps:

1. **Run Unit Tests (Python)**
   ```bash
   pytest services/api/tests -v --cov=services/api --cov-report=term-missing
   ```
   - All tests must pass (exit code 0)
   - Coverage must not decrease: target >80% for backend
   - If test fails: fix the code, don't skip the test

2. **Run Unit Tests (TypeScript/JavaScript)**
   ```bash
   npm run test:unit -- --coverage uis/backoffice
   ```
   - All tests must pass
   - Coverage must not decrease: target >70% for frontend
   - Run in watch mode during development, full run before commit

3. **Run Integration Tests (Python)**
   ```bash
   pytest services/api/tests/integration -v
   ```
   - Tests that touch database, APIs, external services
   - Must pass in full (not individual tests)

4. **Run E2E Tests (TypeScript, if applicable)**
   ```bash
   npm run test:e2e -- --headless uis/backoffice
   ```
   - Critical user flows only (appointment booking, patient record access)
   - Can run in CI before merge, but verify locally if you modified UI flows

**Coverage Requirements by Component:**

| Component | Min Coverage | Type | Requirement |
|---|---|---|---|
| Backend API | 80% | Line coverage | No decrease allowed |
| Backend Models | 85% | Line coverage | No decrease allowed |
| Frontend Components | 70% | Line coverage | No decrease allowed |
| Frontend Pages | 60% | Line coverage | No decrease allowed |

**Checklist:**
- [ ] All unit tests pass (Python)
- [ ] All unit tests pass (TypeScript)
- [ ] All integration tests pass
- [ ] Code coverage not decreased
- [ ] No skipped tests (`@skip`, `xit()`, `pytest.mark.skip`)
- [ ] New code has >80% test coverage

**If Any Test Fails:**
1. Read the error message carefully
2. Debug the failure (use IDE breakpoints, add print statements)
3. Fix the code (not the test, unless test is wrong)
4. Verify fix by running test again
5. If multiple failures, check if you broke dependencies

**Definition of Done for Step 2:**
```
✅ All unit tests pass
✅ All integration tests pass
✅ All E2E tests pass
✅ Coverage maintained or improved
✅ No test skips or TODOs in test code
```

---

### **STEP 3: Compliance & Security Checks** (5-15 min)

**Purpose:** Verify code meets HIPAA/GDPR and security standards.

#### Sub-steps:

1. **Compliance Checklist (Manual Review)**
   
   Before every commit, ask yourself:
   
   - [ ] **Does this code touch patient data?**
     - YES → Verify encryption at rest ✅
     - YES → Verify encryption in transit (TLS) ✅
     - YES → Verify audit logging implemented ✅
     - YES → Reference `techContext.md` Section 6 ✅
   
   - [ ] **Does this code create an API endpoint?**
     - YES → Verify authentication required ✅
     - YES → Verify authorization checks (RBAC) ✅
     - YES → Verify clinic-scoped filtering ✅
     - YES → Verify error messages don't leak PHI ✅
   
   - [ ] **Does this code write to database?**
     - YES → Verify clinic_id is included in every query ✅
     - YES → Verify data residency rules (USA vs UK) ✅
     - YES → Verify no raw patient names in logs ✅
   
   - [ ] **Does this code handle authentication/passwords?**
     - YES → Verify passwords hashed (bcrypt, Argon2) ✅
     - YES → Verify no credentials in code/logs ✅
     - YES → Verify JWT tokens have expiration ✅
   
   - [ ] **Does this code use third-party APIs/libraries?**
     - YES → Verify Business Associate Agreement (BAA) exists ✅
     - YES → Verify vendor has HIPAA/GDPR compliance docs ✅
     - YES → Verify dependency is not in git .gitignore (secrets) ✅

2. **Automated Security Scan (Bandit for Python)**
   ```bash
   bandit -r services/api --severity-level medium
   ```
   - Checks for hardcoded passwords, SQL injection, insecure crypto
   - Zero high-severity findings allowed
   - Medium findings must have documented justification

3. **Dependency Check**
   ```bash
   safety check services/api/requirements.txt
   pip-audit
   npm audit uis/backoffice
   ```
   - Verifies no known vulnerable dependencies
   - If vulnerable: either update or escalate to Security Lead

4. **Secrets Scan (Git Hooks)**
   ```bash
   git secrets --scan
   ```
   - Automatically runs pre-commit
   - Catches API keys, passwords, tokens accidentally committed
   - If triggered: immediately revoke exposed credentials

**Checklist:**
- [ ] Code touching PHI has encryption checks
- [ ] All APIs have auth/authz checks
- [ ] Database queries include clinic_id filtering
- [ ] No credentials in code or logs
- [ ] Bandit security scan passed
- [ ] No new vulnerable dependencies
- [ ] Git secrets scan passed
- [ ] All findings documented (if any)

**If Any Security Check Fails:**
1. Review the security finding
2. If it's a false positive: add `# noqa: B101` comment with justification
3. If it's real: fix immediately (don't commit with security issues)
4. Never suppress security warnings without approval from Security Lead

**Definition of Done for Step 3:**
```
✅ Compliance checklist completed (all YES items checked)
✅ No patient data unencrypted
✅ All APIs have auth/authz
✅ No clinic data leaked across regions
✅ Bandit scan passed
✅ Dependency audit passed
✅ No secrets in code
```

---

### **STEP 4: Code Review & Documentation** (10-30 min)

**Purpose:** Ensure code is understandable and aligns with project standards.

#### Sub-steps:

1. **Self-Review (Code Walkthrough)**
   
   Before creating PR, review your own code:
   
   - [ ] Does my code match the architecture in `techContext.md`?
   - [ ] Did I follow naming conventions (snake_case Python, camelCase TS)?
   - [ ] Is my code DRY (Don't Repeat Yourself)? Any duplicates?
   - [ ] Are error messages helpful (without leaking PHI)?
   - [ ] Did I add docstrings/comments for complex logic?
   - [ ] Is my PR description clear and links the GitHub issue?

2. **Documentation Updates**
   
   If you modified:
   - [ ] API endpoint → Update OpenAPI docstring (FastAPI auto-generates Swagger)
   - [ ] Database schema → Update `docs/DATABASE.md`
   - [ ] Configuration → Update `docs/CONFIGURATION.md`
   - [ ] Deployment → Update `docs/DEPLOYMENT.md`
   - [ ] Architecture → Update `docs/ARCHITECTURE.md`
   - [ ] New feature → Add to `README.md` of that component

3. **Commit Message Format**
   
   Use Conventional Commits format:
   ```
   type(scope): subject
   
   body (optional)
   
   footer (optional)
   
   Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
   ```
   
   **Types:**
   - `feat` — New feature
   - `fix` — Bug fix
   - `refactor` — Code restructuring (no behavior change)
   - `docs` — Documentation only
   - `test` — Tests only
   - `perf` — Performance improvement
   - `chore` — Dependencies, config, etc.
   
   **Scope:** Component (e.g., `auth`, `patients`, `appointments`, `ui`)
   
   **Examples:**
   ```
   feat(appointments): add no-show prediction to booking endpoint
   
   - Integrated scikit-learn model for no-show risk scoring
   - Added risk score to appointment response (0-1 scale)
   - Logged predictions for audit trail per HIPAA requirements
   - Covered with unit and integration tests (95% coverage)
   
   Closes #123
   Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
   ```
   
   ```
   fix(auth): prevent session hijacking with secure cookie flags
   
   - Set HttpOnly and Secure flags on session cookies
   - Added SameSite=Strict for CSRF protection
   - Verified with security scan (Bandit)
   
   Closes #456
   Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
   ```

4. **Create Pull Request with Template**
   
   Every PR must include:
   ```markdown
   ## Description
   [What this PR does and why]
   
   ## Related Issue
   Closes #[issue number]
   
   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Breaking change
   - [ ] Documentation update
   
   ## Testing Done
   - [x] Unit tests added/updated (coverage: 92%)
   - [x] Integration tests pass
   - [x] E2E tests pass (if UI changed)
   - [x] Manual testing on staging
   
   ## Compliance Checklist
   - [x] Code touching PHI has encryption
   - [x] All APIs have auth/authz
   - [x] Audit logging implemented
   - [x] Data residency rules followed
   - [x] No hardcoded credentials
   - [x] Security scan passed
   
   ## Memory Bank References
   - `projectbrief.md` → [relevant section]
   - `techContext.md` → [relevant section]
   - `progress.md` → [sprint/task reference]
   
   ## Screenshots (if UI changed)
   [Before/after screenshots or GIFs]
   
   ## Notes for Reviewer
   [Anything reviewer should pay special attention to]
   ```

5. **Request Review from Peers**
   
   Before merging, get:
   - [ ] 1 approval from team member (same role or complementary)
   - [ ] 1 approval from Tech Lead (for architecture/design)
   - [ ] Security Lead approval (if code touches PHI or auth)
   - [ ] Product Manager approval (if changes user-facing behavior)
   
   **Who can approve?**
   - Backend code → Backend engineer + Tech Lead
   - Frontend code → Frontend engineer + Tech Lead
   - ML model → ML engineer + Tech Lead
   - Infrastructure → DevOps + Tech Lead
   - Anything → Tech Lead (always required)

6. **Address Review Comments**
   
   - [ ] Reviewer approved without changes
   - [ ] OR Reviewer requested changes → Make changes, re-request review
   - [ ] OR Conversation started → Discuss in PR comments or Slack
   - Never push commits after approval (creates new revision to review)

**Checklist:**
- [ ] Self-review completed
- [ ] Documentation updated (if needed)
- [ ] Commit message follows Conventional Commits
- [ ] PR description complete with template
- [ ] All relevant memory bank files referenced in PR
- [ ] At least 2 approvals (1 peer + 1 lead)
- [ ] All comments resolved
- [ ] CI pipeline passes
- [ ] No "merge conflicts" (rebase if needed)

**If Code Review Fails:**
1. Read reviewer comments carefully
2. Discuss in PR if unclear
3. Make requested changes in new commits (don't amend history)
4. Re-request review
5. Don't merge until satisfied with feedback

**Definition of Done for Step 4:**
```
✅ Self-review completed
✅ Documentation updated
✅ Commit message formatted correctly
✅ PR created with full template
✅ 2 approvals received (peer + lead)
✅ All CI checks pass
✅ No unresolved comments
```

---

## 🔄 Complete Workflow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│          Developer Writes Code on Feature Branch             │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│         STEP 1: Code Quality & Type Safety (5-10 min)       │
│  ✓ black (Python formatting)                                │
│  ✓ ruff (Python linting)                                    │
│  ✓ prettier (TypeScript formatting)                         │
│  ✓ eslint (TypeScript linting)                              │
│  ✓ mypy (Python type checking)                              │
│  ✓ tsc (TypeScript type checking)                           │
└─────────────────────────────────────────────────────────────┘
   │ ALL PASS ✓              │ FAIL ✗
   ↓                         ↓
(continue)            Fix code, go back
   ↓                 to formatting/linting
┌─────────────────────────────────────────────────────────────┐
│           STEP 2: Automated Tests (10-20 min)               │
│  ✓ pytest (Python unit tests, >80% coverage)                │
│  ✓ jest (TypeScript unit tests, >70% coverage)              │
│  ✓ pytest integration (database, APIs)                      │
│  ✓ cypress E2E (critical UI flows, if applicable)           │
└─────────────────────────────────────────────────────────────┘
   │ ALL PASS ✓              │ FAIL ✗
   ↓                         ↓
(continue)            Debug test, fix code
   ↓                 go back to test
┌─────────────────────────────────────────────────────────────┐
│      STEP 3: Compliance & Security (5-15 min)               │
│  ✓ Manual compliance checklist (PHI, auth, clinic_id)       │
│  ✓ bandit (Python security scan)                            │
│  ✓ safety check (Python dependencies)                       │
│  ✓ npm audit (JavaScript dependencies)                      │
│  ✓ git secrets (no credentials leaked)                      │
└─────────────────────────────────────────────────────────────┘
   │ ALL PASS ✓              │ FAIL ✗
   ↓                         ↓
(continue)            Fix security issue
   ↓                 or escalate to
┌─────────────────────────────────────────────────────────────┐
│       STEP 4: Code Review & Documentation (10-30 min)       │
│  ✓ Self-review (code walkthrough)                           │
│  ✓ Update documentation (if needed)                         │
│  ✓ Conventional Commits message                             │
│  ✓ Create PR with full template                             │
│  ✓ Request 2 approvals (peer + tech lead)                   │
│  ✓ Address review comments                                  │
│  ✓ All CI checks pass                                       │
└─────────────────────────────────────────────────────────────┘
   │ APPROVED ✓              │ REJECTED ✗
   ↓                         ↓
(continue)            Address feedback
   ↓                 go back to code
┌─────────────────────────────────────────────────────────────┐
│                    MERGE TO DEVELOP                         │
│  Squash commits (1 commit per feature)                      │
│  Auto-delete feature branch                                 │
│  Trigger CI/CD pipeline (staging deployment)                │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Local Pre-Commit Hook Setup

**Automate Steps 1-3 locally (before you even commit):**

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
set -e

echo "🔍 Running pre-commit checks..."

# STEP 1: Code Quality
echo "  → Formatting Python code with black..."
black services/api --line-length 100 || exit 1

echo "  → Linting Python code with ruff..."
ruff check services/api --fix || exit 1

echo "  → Formatting TypeScript code with prettier..."
prettier uis/backoffice/src --write || exit 1

echo "  → Linting TypeScript code with eslint..."
eslint uis/backoffice/src --fix || exit 1

echo "  → Type-checking Python code with mypy..."
mypy services/api || exit 1

echo "  → Type-checking TypeScript code with tsc..."
tsc --noEmit uis/backoffice || exit 1

# STEP 2: Tests
echo "  → Running Python unit tests..."
pytest services/api/tests -v --cov=services/api --cov-fail-under=80 || exit 1

echo "  → Running TypeScript unit tests..."
npm run test:unit -- --coverage uis/backoffice || exit 1

# STEP 3: Security
echo "  → Running Bandit security scan..."
bandit -r services/api --severity-level medium || exit 1

echo "  → Checking for secrets..."
git secrets --scan || exit 1

echo "✅ All pre-commit checks passed!"
```

**Install hook:**
```bash
chmod +x .git/hooks/pre-commit
```

**Now, every `git commit` runs all checks automatically!**

---

## ⚠️ Emergency Override (Last Resort)

**When you need to bypass the workflow:**

This should be **extremely rare**. Only in these cases:
- 🚨 Production outage (patient care at risk)
- 🔒 Security incident (breach in progress)
- 📋 Compliance emergency (regulatory deadline)

**How to override:**
```bash
# Skip pre-commit hook
git commit --no-verify

# IMMEDIATELY after:
1. Add `[SKIP-CHECKS]` to commit message with justification
2. Escalate to James Osei (CTO) in Slack
3. File a GitHub issue to retroactively verify skipped code
4. Run full checks in CI/CD (don't skip those)
```

**Example:**
```
[SKIP-CHECKS] fix(auth): emergency revoke of leaked session tokens

Production incident: Session tokens exposed in log aggregator.
Revoking all tokens immediately to prevent patient data access.

Emergency override justified by HIPAA breach protocol.

Escalated to: James Osei, Security Lead
Ticket: #INCIDENT-2026-001

Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```

---

## 📊 Workflow Checklist (Copy & Paste for Every Commit)

```markdown
## Pre-Commit Checklist

### STEP 1: Code Quality ✓
- [ ] black formatting passed
- [ ] ruff linting passed
- [ ] prettier formatting passed
- [ ] eslint linting passed
- [ ] mypy type checking passed
- [ ] tsc type checking passed

### STEP 2: Tests ✓
- [ ] pytest unit tests passed
- [ ] jest unit tests passed
- [ ] pytest integration tests passed
- [ ] E2E tests passed (if UI changed)
- [ ] Code coverage maintained (>80% backend, >70% frontend)

### STEP 3: Compliance & Security ✓
- [ ] Code touching PHI has encryption
- [ ] All APIs have authentication & authorization
- [ ] Database queries include clinic_id filtering
- [ ] No hardcoded credentials
- [ ] bandit security scan passed
- [ ] npm audit / safety check passed
- [ ] git secrets scan passed

### STEP 4: Code Review & Documentation ✓
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Commit message follows Conventional Commits
- [ ] PR created with full template
- [ ] 2 approvals received (peer + tech lead)
- [ ] All comments resolved
- [ ] CI/CD pipeline passes

### Ready to Merge? ✓
- [ ] All 4 steps completed
- [ ] No unresolved comments
- [ ] CI status is green
- [ ] Rebase on develop (no merge commits)
```

---

## 📞 Escalation Path

If any step fails and you don't know how to fix:

| Step | Failure | Escalate To | Time |
|---|---|---|---|
| 1 | Linting/formatting error | Tech Lead (Slack) | 2 hours |
| 2 | Test failure | Test owner + Tech Lead | 4 hours |
| 3 | Security finding | Security Lead (Claire) | Immediate |
| 4 | Review rejection | PR author + reviewer (discuss) | Same day |

---

## ✅ Sign-Off

By committing to `HealthCore Digital`, you acknowledge:

✅ I have read this workflow document  
✅ I understand all 4 steps are mandatory  
✅ I will not commit without completing all steps  
✅ I will escalate failures, not hide them  
✅ I will follow the memory bank guidance  
✅ I understand HIPAA/GDPR are non-negotiable  

**Last Reviewed:** November 17, 2026  
**Next Review:** February 17, 2027 (Quarterly)  
**Owner:** James Osei (CTO)

# Pre-Commit Checklist — Quick Reference

**Print this or bookmark it!** Use before every commit to HealthCore Digital.

---

## ✅ 4-STEP OBLIGATORY WORKFLOW

### 🔧 STEP 1: Code Quality & Type Safety (5-10 min)

```bash
# Copy & paste these commands:

# Python formatting
black services/api --line-length 100

# Python linting  
ruff check services/api --fix

# TypeScript formatting
prettier uis/backoffice/src --write

# TypeScript linting
eslint uis/backoffice/src --fix

# Python type checking
mypy services/api

# TypeScript type checking
tsc --noEmit uis/backoffice
```

**✓ All passed?** Continue to Step 2  
**✗ Failed?** Fix the error, re-run command

---

### 🧪 STEP 2: Automated Tests (10-20 min)

```bash
# Run all tests:

# Python unit tests (must pass + >80% coverage)
pytest services/api/tests -v --cov=services/api --cov-fail-under=80

# TypeScript unit tests (must pass + >70% coverage)
npm run test:unit -- --coverage uis/backoffice

# Python integration tests
pytest services/api/tests/integration -v

# E2E tests (if UI changed)
npm run test:e2e -- --headless
```

**✓ All passed?** Continue to Step 3  
**✗ Failed?** Debug test failure, fix code, re-run

---

### 🔒 STEP 3: Compliance & Security (5-15 min)

**Manual Checklist (3 min):**
```
Does this code touch patient data?
  ☐ NO → Skip encryption checks
  ☐ YES → ✓ Encryption implemented?
  ☐ YES → ✓ Audit logging implemented?

Does this code create API endpoint?
  ☐ NO → Skip auth checks
  ☐ YES → ✓ Authentication required?
  ☐ YES → ✓ Authorization (RBAC) checked?
  ☐ YES → ✓ Clinic-scoped filtering applied?

Does this code write to database?
  ☐ NO → Skip residency checks
  ☐ YES → ✓ clinic_id included in queries?
  ☐ YES → ✓ USA data ≠ UK data separated?

Does this code handle auth/passwords?
  ☐ NO → Skip password checks
  ☐ YES → ✓ Passwords hashed (bcrypt)?
  ☐ YES → ✓ No credentials in logs?

Does this code use third-party APIs/libs?
  ☐ NO → Skip vendor checks
  ☐ YES → ✓ BAA exists?
  ☐ YES → ✓ Vendor HIPAA/GDPR compliant?
```

**Automated Scans (5 min):**
```bash
# Security scan (must have zero high-severity findings)
bandit -r services/api --severity-level medium

# Check dependencies for vulnerabilities
safety check services/api/requirements.txt
npm audit uis/backoffice

# Scan for accidentally committed secrets
git secrets --scan
```

**✓ All passed?** Continue to Step 4  
**✗ Failed?** Fix security issue or escalate to Security Lead

---

### 👀 STEP 4: Code Review & Documentation (10-30 min)

**Self-Review Checklist (5 min):**
```
☐ Code matches architecture in techContext.md?
☐ Naming follows conventions (snake_case Python, camelCase TS)?
☐ No code duplication (DRY principle)?
☐ Error messages helpful + no PHI leaks?
☐ Complex logic has comments/docstrings?

☐ Documentation updated?
  ☐ API docs (FastAPI docstring) → Swagger auto-generated
  ☐ Database changes → docs/DATABASE.md
  ☐ Config changes → docs/CONFIGURATION.md
  ☐ Deployment → docs/DEPLOYMENT.md
  ☐ Architecture → docs/ARCHITECTURE.md
  ☐ Feature → README.md of component
```

**Commit Message (Conventional Commits):**
```
type(scope): subject line max 50 chars

Body (optional):
- Detailed explanation
- Why this change matters
- Any breaking changes

Closes #123
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `test`, `perf`, `chore`  
**Scopes:** `auth`, `patients`, `appointments`, `ui`, `billing`, `compliance`, etc.

**Example:**
```
feat(appointments): add no-show prediction to booking

- Integrated ML model for risk scoring
- Returns risk_score (0-1) in appointment response
- Logged predictions for audit trail
- 95% test coverage (12 test cases)

Closes #456
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>
```

**Create Pull Request with Template:**

```markdown
## Description
[What and why]

## Related Issue
Closes #[number]

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Docs

## Testing
- [x] Unit tests (92% coverage)
- [x] Integration tests pass
- [x] E2E tests pass
- [x] Manual testing

## Compliance
- [x] PHI encrypted
- [x] Auth/authz implemented
- [x] Audit logging
- [x] Data residency rules
- [x] No credentials in code
- [x] Security scan passed

## Memory Bank
- `projectbrief.md` → [section]
- `techContext.md` → [section]
- `progress.md` → [sprint]

## Reviewer Notes
[Anything special to review]
```

**Request Approvals:**
- [ ] 1 approval from peer (same role)
- [ ] 1 approval from Tech Lead
- [ ] Security Lead (if PHI/auth code)
- [ ] Product Manager (if user-facing)

**Address Comments:**
```bash
# Make changes in new commits (don't amend history)
git add .
git commit -m "refactor(appointments): address review feedback"
git push

# Re-request review when done
```

**✓ 2 approvals + all CI pass?** Tech Lead merges  
**✗ Comments unresolved?** Address feedback, re-request review

---

## ⚡ Fast Commands

```bash
# Check everything at once (run from repo root)
bash scripts/pre-commit.sh

# Or manually run each layer:

# Layer 1: Format & lint
black services/api && ruff check services/api --fix && \
prettier uis/backoffice/src --write && eslint uis/backoffice/src --fix

# Layer 2: Type check
mypy services/api && tsc --noEmit uis/backoffice

# Layer 3: Tests
pytest services/api/tests --cov-fail-under=80 && \
npm run test:unit -- --coverage uis/backoffice

# Layer 4: Security
bandit -r services/api --severity-level medium && \
git secrets --scan
```

---

## 🚨 If Anything Fails

| Failure | Action |
|---------|--------|
| Format/lint error | Fix code, re-run |
| Type error | Add type annotation, re-run |
| Test failure | Debug with IDE, fix code, re-run |
| Security warning | Fix issue OR escalate to Security Lead |
| Review rejection | Discuss in PR comments, make changes, re-request |

---

## 🛑 Emergency Override

**ONLY in production emergencies:**
```bash
git commit --no-verify -m "[SKIP-CHECKS] your message"
```

Then immediately:
1. Notify James Osei in Slack
2. File GitHub issue to retroactively verify
3. Run full checks in CI/CD

---

## 📌 Remember

- **ALL 4 STEPS ARE MANDATORY**
- **No skipping tests**
- **No hardcoded credentials**
- **No patient data unencrypted**
- **Audit log every patient access**
- **Data residency rules non-negotiable**

---

## 📞 Help

**Still confused?** Read in this order:
1. `docs/COMMIT_WORKFLOW.md` (full workflow)
2. `CONTRIBUTING.md` (dev guide)
3. Slack #healthcore-dev (ask team)
4. James Osei (CTO)

---

**Version:** 1.0 | **Last Updated:** November 17, 2026 | **Owner:** James Osei (CTO)

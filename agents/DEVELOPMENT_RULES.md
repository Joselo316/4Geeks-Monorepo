# 🛠️ Agent Development Rules — HealthCore Digital

**Authoritative development rules for all agents working in HealthCore Digital**

**Version:** 1.0 | **Date:** November 17, 2026 | **Owner:** James Osei (CTO)

---

## 📋 Table of Contents

1. [Rule 1: Code Quality Standards](#rule-1-code-quality-standards)
2. [Rule 2: Security & Compliance First](#rule-2-security--compliance-first)
3. [Rule 3: Patient Data Protection](#rule-3-patient-data-protection)
4. [Rule 4: Testing Requirements](#rule-4-testing-requirements)
5. [Rule 5: Documentation Discipline](#rule-5-documentation-discipline)
6. [Rule 6: File Protection Respect](#rule-6-file-protection-respect)
7. [Rule 7: Collaboration Protocol](#rule-7-collaboration-protocol)
8. [Scope of Application](#scope-of-application)
9. [Enforcement & Violations](#enforcement--violations)
10. [Quick Reference](#quick-reference)

---

## Rule 1: Code Quality Standards

**Scope:** ALL code changes in HealthCore Digital (Python, TypeScript, JavaScript)

### Requirements

```
✓ MANDATORY CODE QUALITY CHECKS:
  ├─ Python:
  │  ├─ black --line-length 100 (auto-formatting)
  │  ├─ ruff check --fix (linting with auto-fix)
  │  ├─ mypy (type checking, 0 errors)
  │  └─ pylint (optional, for complex code)
  │
  ├─ TypeScript/JavaScript:
  │  ├─ prettier --write (auto-formatting)
  │  ├─ eslint --fix (linting with auto-fix)
  │  ├─ tsc --noEmit (type checking)
  │  └─ TSDoc comments for public APIs
  │
  └─ ALL COMMITS:
     ├─ No warnings in linting output
     ├─ No type errors (mypy 0 errors, tsc passes)
     ├─ No code formatting issues
     └─ Conventional Commits format mandatory

✗ VIOLATIONS:
  ❌ Committing code with linting errors
  ❌ Pushing code with type errors
  ❌ Using deprecated APIs/functions
  ❌ Code that doesn't follow language style guide
  ❌ Using `any` type without explicit justification comment
  ❌ Commented-out code in final commits
```

### Application Scope

- **Where:** `services/api/`, `uis/backoffice/`, `agents/`, `packages/shared/`
- **Who:** All agents (backend-dev, frontend-dev, ml-engineer)
- **When:** BEFORE every commit (STEP 1 of 4-step workflow)
- **Tools:** Pre-commit hooks or manual checks before git commit

### Enforcement Point

```bash
# STEP 1: Code Quality & Type Safety (5-10 min)
# This is the first check in COMMIT_WORKFLOW.md

# Python
black services/api --line-length 100
ruff check services/api --fix
mypy services/api --strict

# TypeScript
prettier --write uis/backoffice/src
eslint uis/backoffice/src --fix
tsc --noEmit

# If ANY tool fails → STOP, fix code, re-run tools
# Only proceed to STEP 2 when all tools pass
```

---

## Rule 2: Security & Compliance First

**Scope:** ALL code touching patient data, authentication, or sensitive information

### Requirements

```
✓ MANDATORY SECURITY CHECKS:

Before writing ANY code that touches:
├─ Patient data (PHI)
├─ Authentication/authorization
├─ API endpoints
├─ Database queries
└─ Third-party integrations

You MUST:
├─ Read techContext.md → "Security & Compliance" section
├─ Read memory-bank/progress.md → "Compliance Checkpoints"
├─ Implement HIPAA-compliant code
├─ Implement UK GDPR-compliant code
├─ Add audit logging for PHI access
├─ Encrypt sensitive data at rest
└─ Validate all inputs (never trust user input)

✗ VIOLATIONS (Will block PR merge):
  ❌ Patient data without encryption at rest
  ❌ PHI access without audit logging
  ❌ Authentication bypass or weak auth
  ❌ Hardcoded credentials/secrets
  ❌ SQL injection vulnerability
  ❌ Cross-site scripting (XSS) vulnerability
  ❌ No input validation/sanitization
  ❌ Storing passwords in plaintext
  ❌ Logging patient data to unencrypted files
  ❌ Direct database access without clinic_id filter
```

### Application Scope

- **Where:** `services/api/models/`, `services/api/schemas/`, `services/api/routers/`, `uis/backoffice/pages/`
- **Who:** All agents (backend-dev, frontend-dev, ml-engineer, security-lead)
- **When:** During code review (STEP 4 of 4-step workflow)
- **Tools:** Manual checklist + bandit (Python) + npm audit (Node)

### Enforcement Point

```
STEP 3: Compliance & Security (5-15 min)

Security Scan Checklist:
☐ No patient data unencrypted
☐ All APIs have authentication
☐ All DB queries filtered by clinic_id
☐ No hardcoded credentials
☐ Audit logging implemented (if PHI touched)
☐ Errors don't leak PHI
☐ Bandit scan passed (Python)
☐ npm audit passed (Node)
☐ No SQL injection vulnerabilities
☐ No XSS vulnerabilities

Run:
bandit -r services/api --severity-level medium
npm audit --audit-level=moderate

If FAILURES: STOP, fix security issues, re-run checks
```

---

## Rule 3: Patient Data Protection

**Scope:** ALL code handling, storing, transmitting, or logging patient information

### Requirements

```
✓ MANDATORY DATA PROTECTION:

Data Residency (CRITICAL):
├─ USA patient data → USA database only
├─ UK patient data → UK database only
├─ Every query MUST include clinic_id filter
└─ No cross-residency data queries

Encryption:
├─ At REST: AES-256 for all PHI
├─ In TRANSIT: TLS 1.3 for all APIs
├─ In DATABASE: Encrypted columns for SSN, DOB, medical records
└─ Backup files encrypted

Access Control:
├─ RBAC enforced (Role-Based Access Control)
├─ Clinic-specific data isolation
├─ Audit log of all PHI access
├─ User can only see their own clinic's data
└─ Admin access is logged and limited

Data Retention:
├─ Follow HIPAA retention policy (6-10 years typically)
├─ Implement data deletion/anonymization process
├─ Document retention schedule
└─ Archive encrypted backups

✗ CRITICAL VIOLATIONS (Security incident):
  🚨 Patient data crosses data residency boundary
  🚨 Query doesn't filter by clinic_id
  🚨 Unencrypted PHI in database
  🚨 PHI logged to unencrypted files
  🚨 Patient data exposed via error message
  🚨 Backup files not encrypted
  🚨 Access control bypassed
  🚨 User sees another clinic's data
```

### Application Scope

- **Where:** `services/api/models/patient.py`, `services/api/routers/patients.py`, `uis/backoffice/pages/patients/`
- **Who:** backend-dev, ml-engineer (data scientists)
- **When:** Code review and security review
- **Tools:** Manual security review + bandit + data flow analysis

### Enforcement Point

```
Security Review Checklist:
☐ Verified data residency (USA/UK separation)
☐ All queries include clinic_id filter
☐ PHI is encrypted at rest
☐ No PHI in logs/error messages
☐ Access control verified
☐ Audit logging implemented
☐ Backup encryption confirmed
☐ HIPAA compliance verified
☐ UK GDPR compliance verified

FAIL ANY CHECK → PR BLOCKED until fixed
```

---

## Rule 4: Testing Requirements

**Scope:** ALL code changes must have corresponding tests

### Requirements

```
✓ MANDATORY TEST COVERAGE:

Backend (Python/FastAPI):
├─ Unit tests: pytest (>80% coverage minimum)
├─ Integration tests: test database interactions
├─ End-to-end tests: full workflow testing
├─ Security tests: test auth/RBAC/data isolation
└─ Compliance tests: test HIPAA/GDPR requirements

Frontend (React/TypeScript):
├─ Unit tests: jest (>70% coverage minimum)
├─ Component tests: test UI behavior
├─ Integration tests: test API communication
├─ E2E tests: test user workflows
└─ Accessibility tests: test a11y compliance

✗ VIOLATIONS:
  ❌ Code change without tests
  ❌ Tests exist but don't pass
  ❌ Coverage below 80% (backend) or 70% (frontend)
  ❌ Tests don't cover security scenarios
  ❌ Tests don't cover error cases
  ❌ Tests are commented out
  ❌ Tests are skipped (pending)
  ❌ No test for bug fix (causes regression)
```

### Application Scope

- **Where:** `services/api/tests/`, `uis/backoffice/src/__tests__/`
- **Who:** All agents (backend-dev, frontend-dev, qa-engineer)
- **When:** STEP 2 of 4-step workflow (before commit)
- **Tools:** pytest, jest, coverage reports

### Enforcement Point

```
STEP 2: Automated Tests (10-20 min)

Required to run:
pytest services/api/tests -v --cov=services/api --cov-report=term-missing
npm run test:unit -- --coverage uis/backoffice

PASS CRITERIA:
├─ All tests pass (0 failures)
├─ Coverage ≥ 80% (backend)
├─ Coverage ≥ 70% (frontend)
├─ No skipped/pending tests (unless documented)
└─ Security tests included

FAIL ANY → STOP, fix code/tests, re-run
Only proceed to STEP 3 when all tests pass
```

---

## Rule 5: Documentation Discipline

**Scope:** ALL code changes must include documentation updates

### Requirements

```
✓ MANDATORY DOCUMENTATION:

Code Documentation:
├─ Docstrings for all functions/methods (Python)
├─ JSDoc comments for all public APIs (TypeScript)
├─ Comments for complex logic
├─ Type annotations on all function signatures
├─ Parameter descriptions with types
└─ Return value descriptions

README Updates:
├─ Update README.md if behavior changed
├─ Document new features with examples
├─ Document breaking changes
├─ Document migration steps (if applicable)
└─ Update table of contents if structure changed

API Documentation:
├─ Update OpenAPI/Swagger documentation
├─ Document new endpoints
├─ Document request/response schemas
├─ Document error responses
└─ Document authentication requirements

Commit Messages:
├─ Format: type(scope): subject
├─ type: feat, fix, docs, style, refactor, test, chore
├─ scope: specific area (e.g., patient-api, dashboard)
├─ subject: short description (<50 chars)
├─ body: detailed explanation (why, not what)
├─ footer: reference issues (Closes #123)
└─ Example: "feat(patient-api): Add patient search endpoint"

✗ VIOLATIONS:
  ❌ Code change without docstrings
  ❌ Public API without documentation
  ❌ Feature added without README update
  ❌ No comments on complex logic
  ❌ Commit message doesn't explain why
  ❌ Breaking changes not documented
  ❌ API endpoint not documented
  ❌ No type annotations
```

### Application Scope

- **Where:** All `.py` and `.ts`/`.tsx` files in `services/`, `uis/`, `agents/`
- **Who:** All agents
- **When:** Before commit (included in STEP 4)
- **Tools:** Manual review + doc generation tools

### Enforcement Point

```
STEP 4: Documentation & Code Review (10-30 min)

Checklist:
☐ Docstrings added/updated
☐ README updated if needed
☐ API docs updated (if endpoint changed)
☐ Commit message follows Conventional Commits
☐ Code comments for complex logic
☐ Type annotations complete
☐ Breaking changes documented

FAIL ANY → Reviewer requests changes in PR
```

---

## Rule 6: File Protection Respect

**Scope:** ALL agents (non-negotiable)

### Requirements

```
✓ MANDATORY FILE PROTECTION:

BEFORE modifying ANY file:
├─ Check its protection level (see docs/PROTECTED_FILES.md)
├─ 🛑 LOCKED? → Ask owner only
├─ 🟠 PROCESS? → Ask Tech Lead (James Osei)
├─ 🟡 CONFIG? → Coordinate with specialist
├─ 🟢 CODE? → Modify if assigned (create PR if not)
├─ 🚫 FORBIDDEN? → NEVER TOUCH

LOCKED Files (Never modify):
├─ memory-bank/projectbrief.md
├─ memory-bank/techContext.md
├─ memory-bank/progress.md
└─ memory-bank/AGENTS.md

PROCESS Files (Ask Tech Lead):
├─ CONTRIBUTING.md
├─ docs/COMMIT_WORKFLOW.md
├─ docs/COMMIT_CHECKLIST.md
└─ docs/WORKFLOW_SUMMARY.md

CONFIG Files (Coordinate):
├─ .github/workflows/*.yml
├─ docker-compose.yml
├─ .flake8, .eslintrc.json, .prettierrc
├─ pyproject.toml, tsconfig.json
└─ jest.config.js

✗ VIOLATIONS (Will cause PR rejection + escalation):
  ❌ Modifying memory-bank/* without owner approval
  ❌ Modifying workflow docs without Tech Lead approval
  ❌ Modifying config files without specialist approval
  ❌ Committing secrets/credentials
  ❌ Modifying another developer's code without PR
```

### Application Scope

- **Where:** Entire repository
- **Who:** ALL agents (mandatory)
- **When:** Before every commit (STEP 1 checklist)
- **Tools:** Manual verification + CI/CD checks (future)

### Enforcement Point

```
Pre-Commit Verification:

Before git commit, verify:
☐ No memory-bank/* files modified (unless you're the owner)
☐ No workflow docs modified (unless approved by James Osei)
☐ No config files modified (unless coordinated)
☐ All changes in services/, uis/, agents/, data/, packages/
☐ OR you have explicit approval documented in PR

If VIOLATED:
→ STOP
→ Revert changes: git revert <commit>
→ Explain in PR why you need exception
→ Get approval from file owner
→ Create new commit with approval reference
```

---

## Rule 7: Collaboration Protocol

**Scope:** ALL agent-to-developer and agent-to-agent interactions

### Requirements

```
✓ MANDATORY COLLABORATION:

When Working with Other Agents:
├─ Check git status (don't modify someone else's work)
├─ Create feature branch (never commit to develop)
├─ Ask before modifying shared code (packages/shared/)
├─ Document any breaking changes
├─ Review other agent's work in PRs
├─ Provide constructive feedback
└─ Help unblock other agents

When Asking for Approval:
├─ Post in #healthcore-dev Slack channel
├─ Provide clear context (what, why, impact)
├─ Link to GitHub issue/PR
├─ Wait for response (don't force changes)
├─ Respect decision (even if you disagree)
└─ Document decision in PR for future reference

When Code Review Comments Received:
├─ Read comments carefully
├─ Don't argue with reviewer (ask for clarification)
├─ Make requested changes
├─ Commit with message: "Address review: [description]"
├─ Re-request review (don't self-approve)
├─ Thank reviewer for feedback
└─ Learn from feedback for next PR

✗ VIOLATIONS:
  ❌ Modifying another agent's work without approval
  ❌ Ignoring code review feedback
  ❌ Self-approving your own PRs
  ❌ Pushing to develop without PR
  ❌ Not responding to Slack messages (24 hours)
  ❌ Arguing with reviewers about code quality
  ❌ Blocking other agents (not sharing status)
  ❌ Breaking someone else's code without fixing it
```

### Application Scope

- **Where:** GitHub PRs, code reviews, Slack communication
- **Who:** All agents
- **When:** Every PR submission and code review cycle
- **Tools:** GitHub PRs, Slack channels (#healthcore-dev)

### Enforcement Point

```
PR Review Standards:

For PRs you submit:
☐ PR has clear title and description
☐ Linked to GitHub issue
☐ All 4-step workflow checks pass
☐ 2 approvals before merge (peer + tech lead)
☐ No "Approve" self-review
☐ All feedback addressed
☐ Commit history is clean

For PRs you review:
☐ Review within 24 hours
☐ Check code quality (all 4 steps)
☐ Check security (no vulnerabilities)
☐ Check tests (coverage requirements)
☐ Check documentation (docstrings, README)
☐ Provide constructive feedback
☐ Approve when all concerns addressed
☐ Don't be a bottleneck (escalate if unsure)
```

---

## Scope of Application

### Who These Rules Apply To

```
✅ APPLIES TO:
├─ All AI agents in HealthCore Digital
├─ backend-dev, frontend-dev, ml-engineer agents
├─ All human developers (enforced in code reviews)
├─ Code changes to HealthCore Digital monorepo
├─ Feature branches before merge to develop
├─ All PRs submitted to any branch
└─ Emergency hotfixes (same rules, documented)

🚫 DOES NOT APPLY TO:
├─ Branches outside HealthCore Digital repo
├─ Local experimentation (before commit)
├─ Archived/deprecated branches
├─ Documentation-only changes (unless major)
└─ Emergency security patches (documented exception)
```

### Where These Rules Apply

```
REPOSITORY: /workspaces/4Geeks-Monorepo/

APPLIES TO ALL THESE DIRECTORIES:
├─ services/api/           (Backend FastAPI code)
├─ services/agents/        (AI Agent tools)
├─ uis/backoffice/         (Frontend React code)
├─ agents/                 (ML/AI components)
├─ packages/shared/        (Shared utilities)
├─ data/                   (Datasets & migrations)
├─ docs/                   (Documentation)
├─ memory-bank/            (Business logic & architecture)
└─ tests/                  (Test files)

EXCLUDES:
├─ .git/                   (Git internals)
├─ node_modules/           (Dependencies)
├─ __pycache__/            (Python cache)
├─ .venv/                  (Virtual environments)
└─ build/dist/             (Build artifacts)
```

### When These Rules Apply

```
✅ ALWAYS APPLY:
├─ When committing code
├─ When submitting PRs
├─ When reviewing code
├─ When working with patient data
├─ When touching security/compliance code
├─ When modifying critical files
└─ When collaborating with other developers

🟡 WITH FLEXIBILITY:
├─ Local experimentation (before commit)
├─ Proof-of-concept code (document as such)
├─ Internal testing branches (cleanup before PR)
└─ Debug/trace code (remove before final commit)

🚫 CANNOT BE BYPASSED:
├─ File protection rules (LOCKED, PROCESS, CONFIG)
├─ Security requirements (encryption, RBAC, audit)
├─ Testing requirements (>80% coverage)
├─ HIPAA/GDPR compliance
├─ Code quality standards (no warnings)
└─ Documentation requirements
```

---

## Enforcement & Violations

### How Rules Are Enforced

```
STEP 1: Prevention (Local)
├─ Agent reads AGENTS.md "Protected Files" section
├─ Agent runs pre-commit checks locally
├─ Agent reviews docs/AGENT_SAFETY_RULES.md
└─ Agent verifies 4-step workflow before commit

STEP 2: Detection (CI/CD)
├─ GitHub Actions runs automated checks:
│  ├─ Linting (black, ruff, prettier, eslint)
│  ├─ Type checking (mypy, tsc)
│  ├─ Tests (pytest, jest)
│  └─ Security scanning (bandit, npm audit)
├─ CI/CD fails if any check doesn't pass
└─ PR blocked until all checks pass

STEP 3: Review (Human)
├─ Tech Lead reviews PR
├─ Checks all 4-step workflow completion
├─ Verifies security/compliance
├─ Checks file protection rules
├─ Verifies documentation
└─ Approves or requests changes

STEP 4: Escalation (Violations)
├─ If rules violated:
│  ├─ PR rejected with explanation
│  ├─ Comment links to relevant rule
│  ├─ Requests agent to fix
│  └─ If repeated: escalate to James Osei
└─ If security violation:
   ├─ PR immediately closed
   ├─ Security team notified
   └─ Investigation + remediation
```

### Violation Types & Consequences

```
LEVEL 1: Code Quality Issue
├─ Violation: Linting error, format error
├─ Detection: Automated (CI/CD)
├─ Consequence: PR blocked, run linting auto-fix
├─ Resolution: Fix locally, re-push
└─ Escalation: None (auto-fixable)

LEVEL 2: Testing Issue
├─ Violation: Tests fail, coverage too low
├─ Detection: Automated (CI/CD)
├─ Consequence: PR blocked, write more tests
├─ Resolution: Add tests, re-run pytest/jest
└─ Escalation: Code review if unsure why test fails

LEVEL 3: Security Issue
├─ Violation: Hardcoded credentials, XSS, SQL injection
├─ Detection: Automated (bandit, npm audit)
├─ Consequence: PR blocked, security review required
├─ Resolution: Fix security issue, security lead approves
└─ Escalation: Security team investigation

LEVEL 4: Compliance Issue
├─ Violation: PHI unencrypted, HIPAA/GDPR violation
├─ Detection: Security review + manual review
├─ Consequence: PR blocked, compliance lead review
├─ Resolution: Implement compliant solution
└─ Escalation: Compliance officer notified

LEVEL 5: File Protection Violation
├─ Violation: Modified memory-bank/* without approval
├─ Detection: Code review + manual check
├─ Consequence: PR rejected, file owner notified
├─ Resolution: Revert, get approval, re-submit
└─ Escalation: James Osei (CTO) if repeated

LEVEL 6: Critical Security Incident
├─ Violation: Secret exposed, patient data breach
├─ Detection: Immediate manual review
├─ Consequence: PR immediately closed, incident response
├─ Resolution: Secret rotation, audit trail review
└─ Escalation: Immediate to Security + CTO + CEO
```

### How to Appeal or Get Exception

```
IF YOU NEED EXCEPTION TO A RULE:

1. Document the reason clearly
   - What rule you need exception for
   - Why you need exception (business/technical reason)
   - What risk you're taking
   - How you'll mitigate risk

2. Post in #healthcore-dev with @james-osei (CTO)
   - Link to relevant rule
   - Provide clear context
   - Propose alternative (if rule is too strict)
   - Wait for approval (minimum 24 hours)

3. Tech Lead reviews & decides
   - Approves exception (documented in PR)
   - Suggests different approach
   - Denies exception with explanation
   - Escalates to CEO if business-critical

4. If approved with exception:
   - Document exception clearly in commit message
   - Add comment in PR explaining exception
   - Create follow-up issue to revisit rule
   - Review exception in next retro
   - Remove exception as soon as possible

✗ NOT ELIGIBLE FOR EXCEPTION:
  ❌ File protection rules (LOCKED, FORBIDDEN)
  ❌ HIPAA/GDPR compliance requirements
  ❌ Security requirements (encryption, auth)
  ❌ Testing coverage minimums (must be met)
  ❌ Code quality standards (black, ruff, etc.)
```

---

## Quick Reference

### Pre-Commit Checklist (Before Every git commit)

```bash
# STEP 1: Code Quality & Type Safety
☐ black services/api --line-length 100
☐ ruff check services/api --fix
☐ mypy services/api --strict
☐ prettier --write uis/backoffice/src
☐ eslint uis/backoffice/src --fix
☐ tsc --noEmit

# STEP 2: Automated Tests
☐ pytest services/api/tests -v --cov-fail-under=80
☐ npm run test:unit -- --coverage

# STEP 3: Compliance & Security
☐ bandit -r services/api --severity-level medium
☐ npm audit --audit-level=moderate
☐ git secrets --scan
☐ Manual security checklist (see docs/AGENT_SAFETY_RULES.md)

# STEP 4: Code Review & Documentation
☐ Self-review code for logic errors
☐ Update README.md if needed
☐ Commit message follows Conventional Commits
☐ Create PR with full description
☐ Request 2 approvals (peer + tech lead)

# BEFORE COMMIT: Verify File Protection
☐ No memory-bank/* files modified (unless owner)
☐ No workflow/config files modified (unless approved)
☐ All changes in permitted directories (services/, uis/, agents/, etc.)
```

### File Protection Quick Lookup

```
Is file in memory-bank/?
├─ YES → 🛑 LOCKED, ask owner
└─ NO → Go to Q2

Is file in docs/ or CONTRIBUTING.md?
├─ YES → 🟠 PROCESS, ask James Osei
└─ NO → Go to Q3

Is file a config (.yml, .json, .toml, .js)?
├─ YES → 🟡 CONFIG, coordinate with specialist
└─ NO → Go to Q4

Is file code in services/, uis/, agents/, packages/, data/?
├─ YES → 🟢 CODE, modify if assigned
└─ NO → Go to Q5

Is it in .git/ or a secret/credential?
├─ YES → 🚫 FORBIDDEN, NEVER TOUCH
└─ NO → Safe to modify
```

### Contacts for Approvals

```
Need approval for...?       Contact                Via
─────────────────────────────────────────────────────
memory-bank/* files        File Owner             Slack
CONTRIBUTING.md            James Osei (CTO)       Slack DM
Linting/CI config         James Osei (CTO)       Slack DM
docker-compose.yml        DevOps Engineer        Slack #healthcore-dev
Shared types             Tech Lead              Slack #healthcore-dev
Feature code (assigned)   Sprint Lead            PR Review
Security/red flag        Security Team          #healthcore-dev URGENT
```

---

## Document Information

- **Title:** Agent Development Rules — HealthCore Digital
- **Version:** 1.0
- **Date Created:** November 17, 2026
- **Last Updated:** November 17, 2026
- **Owner:** James Osei (CTO)
- **Reviewers:** Security Team, Tech Lead, Development Team
- **Status:** ACTIVE & MANDATORY
- **Location:** `/workspaces/4Geeks-Monorepo/agents/DEVELOPMENT_RULES.md`

### How This Document Is Used

```
AT SESSION START (Agents):
├─ Read Section 1-7 (all rules)
├─ Understand scope of application
├─ Review quick reference before first commit
└─ Bookmark for daily reference

BEFORE EVERY COMMIT (Agents):
├─ Review quick reference checklist
├─ Verify 4-step workflow completion
├─ Check file protection rules
└─ Ensure compliance with all rules

IN CODE REVIEWS (Developers):
├─ Reference specific rule if violation found
├─ Link to enforcement point
├─ Explain why rule matters
├─ Provide link to full rule documentation
└─ Request changes to meet rule requirement

DURING ONBOARDING (New Agents):
├─ Read full document (all 10 sections)
├─ Ask questions in #healthcore-dev
├─ Run through mock PR submission
├─ Get feedback from Tech Lead
└─ Confirm understanding before first commit
```

---

**All agents must read and agree to follow these development rules before submitting their first PR to HealthCore Digital.**

**Questions? Ask in #healthcore-dev Slack or contact James Osei (CTO).**


# HealthCore Digital — Complete Workflow Summary

**All documentation you need for development → commit → merge** 

---

## 📚 Documentation Map

### 🔴 **MUST READ BEFORE STARTING**

| File | Purpose | Read Time |
|------|---------|-----------|
| [`memory-bank/projectbrief.md`](/workspaces/4Geeks-Monorepo/memory-bank/projectbrief.md) | Business context, stakeholders, success metrics | 15 min |
| [`memory-bank/techContext.md`](/workspaces/4Geeks-Monorepo/memory-bank/techContext.md) | Technology stack, architecture decisions, compliance | 20 min |
| [`memory-bank/AGENTS.md`](/workspaces/4Geeks-Monorepo/memory-bank/AGENTS.md) | Your role, responsibilities, red flags | 15 min |

**Total: 50 minutes to onboard** ✓

---

### 🔐 **MUST READ: FILE PROTECTION RULES** (Mandatory for Agents)

**Before modifying ANY file, agents MUST read these:**

| File | Purpose | Read Time | Why |
|------|---------|-----------|-----|
| [`docs/AGENT_SAFETY_RULES.md`](/workspaces/4Geeks-Monorepo/docs/AGENT_SAFETY_RULES.md) | Pre-commit safety checklist, red flags | 10 min | **MANDATORY** - Use before every commit |
| [`docs/PROTECTED_FILES.md`](/workspaces/4Geeks-Monorepo/docs/PROTECTED_FILES.md) | Quick reference for protected files | 5 min | Quick lookup of what you can/cannot modify |
| [`docs/FILE_PROTECTION_MATRIX.md`](/workspaces/4Geeks-Monorepo/docs/FILE_PROTECTION_MATRIX.md) | Visual guide with examples | 10 min | Visual learners - see decision flows |

**Total: 25 minutes to understand protections** ✓

**🛑 KEY POINT:** Agents cannot modify `memory-bank/`, workflow docs, config files, or `.git/` without explicit approval from file owner. Violations will result in PR rejection.

---

### 🟢 **BEFORE EACH COMMIT** (Read in this order)

| File | Purpose | Read Time | Use |
|------|---------|-----------|-----|
| [`CONTRIBUTING.md`](/workspaces/4Geeks-Monorepo/CONTRIBUTING.md) | Dev setup, branch naming, testing guide | 20 min | First time only |
| [`docs/COMMIT_WORKFLOW.md`](/workspaces/4Geeks-Monorepo/docs/COMMIT_WORKFLOW.md) | Full 4-step commit workflow with examples | 30 min | Deep dive, understand why |
| [`docs/COMMIT_CHECKLIST.md`](/workspaces/4Geeks-Monorepo/docs/COMMIT_CHECKLIST.md) | Quick reference checklist (copy & paste) | 2 min | **Use before EVERY commit** |
| [`docs/AGENT_SAFETY_RULES.md`](/workspaces/4Geeks-Monorepo/docs/AGENT_SAFETY_RULES.md) | Pre-commit verification (red flags) | 2 min | **Use before EVERY commit** |

**Before each commit: 4 minutes** ✓ (after first read)

---

## ⚡ The 4-Step Workflow (OBLIGATORY)

```
Every developer must complete these 4 steps in order before committing:

┌──────────────────────────────────────────────────────────────┐
│ STEP 1: Code Quality & Type Safety (5-10 min)               │
│  → black, ruff, prettier, eslint, mypy, tsc                 │
│  → No code quality issues allowed                           │
├──────────────────────────────────────────────────────────────┤
│ STEP 2: Automated Tests (10-20 min)                         │
│  → pytest (>80% backend), jest (>70% frontend)              │
│  → All tests must pass                                      │
├──────────────────────────────────────────────────────────────┤
│ STEP 3: Compliance & Security (5-15 min)                    │
│  → Manual compliance checklist                              │
│  → bandit, safety check, npm audit, git secrets             │
│  → Zero security violations allowed                         │
├──────────────────────────────────────────────────────────────┤
│ STEP 4: Code Review & Documentation (10-30 min)             │
│  → Self-review, update docs, commit message                 │
│  → Create PR, get 2 approvals (peer + tech lead)            │
│  → Merge when all CI checks pass                            │
└──────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start for First Commit

**Follow this sequence:**

1. **Read Memory Bank (50 min)**
   ```bash
   cat memory-bank/projectbrief.md
   cat memory-bank/techContext.md
   cat memory-bank/AGENTS.md
   ```

2. **Setup Local Dev (20 min)**
   ```bash
   git clone https://github.com/Joselo316/4Geeks-Monorepo.git
   cd 4Geeks-Monorepo
   docker-compose up
   ```

3. **Read Contributing Guide (20 min)**
   ```bash
   cat CONTRIBUTING.md
   ```

4. **Create Feature Branch**
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/your-feature-name
   ```

5. **Write Code + Tests**
   ```bash
   # Edit files, write tests as you go
   # Commit frequently with Conventional Commits
   ```

6. **Before Final Commit: Run 4-Step Workflow (30 min)**
   ```bash
   # Use docs/COMMIT_CHECKLIST.md as guide
   
   # STEP 1: Format & Lint
   black services/api --line-length 100
   ruff check services/api --fix
   # ... (see COMMIT_CHECKLIST.md for all commands)
   
   # STEP 2: Tests
   pytest services/api/tests -v --cov-fail-under=80
   npm run test:unit -- --coverage uis/backoffice
   
   # STEP 3: Security
   bandit -r services/api --severity-level medium
   git secrets --scan
   
   # STEP 4: Review & Merge
   # Push, create PR, get approvals
   ```

7. **Create PR**
   ```bash
   git push origin feature/your-feature-name
   # Create PR on GitHub with full template
   # Request 2 approvals
   ```

8. **Tech Lead Merges**
   ```bash
   # Squash + merge to develop
   # Auto-deploy to staging
   ```

---

## 📋 Workflow Diagram

```
Write Code
    ↓
Run STEP 1: Code Quality
    ↓ (all pass?)
Run STEP 2: Tests
    ↓ (all pass?)
Run STEP 3: Security
    ↓ (all pass?)
Run STEP 4: Code Review
    ↓ (2 approvals?)
MERGE TO DEVELOP
    ↓
AUTO-DEPLOY TO STAGING
```

---

## ✅ Files You Should Know

### Configuration Files
- `.flake8` — Python linting config
- `.pylintrc` — Python linting config
- `.prettierrc` — TypeScript formatting config
- `.eslintrc.json` — TypeScript linting config
- `pyproject.toml` — Python project config (black, mypy)
- `tsconfig.json` — TypeScript config
- `jest.config.js` — Jest testing config

### CI/CD
- `.github/workflows/` — GitHub Actions workflows
  - `lint.yml` — Automated linting checks
  - `test.yml` — Automated test runs
  - `security.yml` — Security scans

### Scripts
- `scripts/pre-commit.sh` — Run all checks at once

---

## 🔐 Security Reminders

**Before every commit, verify:**

```
☐ No patient data unencrypted
☐ All APIs have authentication
☐ All database queries filtered by clinic_id
☐ No hardcoded credentials
☐ No tokens/keys in code
☐ Audit logging implemented (if PHI touched)
☐ Error messages don't leak PHI
☐ Security scan passed (bandit)
```

---

## 🆘 Troubleshooting

### "My linting checks fail"
```bash
# See COMMIT_CHECKLIST.md Step 1
# Run: black, ruff, prettier, eslint
# They auto-fix most issues
```

### "My tests fail"
```bash
# See COMMIT_CHECKLIST.md Step 2
# Debug with IDE
# Fix code (not test)
# Re-run pytest/jest
```

### "Security scan failed"
```bash
# See COMMIT_CHECKLIST.md Step 3
# Likely hardcoded credential or vulnerable dependency
# Escalate to Security Lead if unsure
```

### "Code review rejected"
```bash
# Read reviewer comments carefully
# Make changes in new commits
# Re-request review
# Don't merge until satisfied
```

---

## 📞 Who to Ask

| Question | Contact |
|----------|---------|
| "What is the business context?" | Read `projectbrief.md` or ask Product Manager |
| "How do I implement X?" | Read `techContext.md` or ask Tech Lead |
| "What's my role?" | Read `AGENTS.md` or ask James Osei |
| "How do I commit?" | Read `COMMIT_WORKFLOW.md` or ask on Slack |
| "Is this secure?" | Read `techContext.md` Security section or ask Security Lead |
| "Is this HIPAA compliant?" | Read `techContext.md` Compliance section or ask Claire Whitfield |
| "Can I modify file X?" | Read `docs/AGENT_SAFETY_RULES.md` or `docs/PROTECTED_FILES.md` |
| "Emergency/blocking issue" | Slack #healthcore-dev or James Osei |

---

## 🛑 Protected Files & What You CAN/CANNOT Modify

**CRITICAL:** Before modifying any file, check its protection level:

```
🛑 LOCKED (Never modify)
   └─ memory-bank/projectbrief.md, techContext.md, progress.md, AGENTS.md
   └─ Owner: Business/Tech Leaders

🟠 PROCESS (Ask Tech Lead)
   └─ CONTRIBUTING.md, docs/COMMIT_*.md, docs/WORKFLOW_SUMMARY.md
   └─ Owner: James Osei (CTO)

🟡 CONFIG (Coordinate first)
   └─ .github/workflows/*.yml, docker-compose.yml, linting config
   └─ Owner: Tech Lead / DevOps Engineer

🟢 CODE (Modify if assigned)
   └─ services/api/, uis/backoffice/, agents/, data/
   └─ Owner: Assigned Sprint/Feature Team

🚫 FORBIDDEN (Never touch)
   └─ .git/, secrets, credentials, unencrypted patient data
   └─ Owner: Security Team
```

**Golden Rule:** "If in doubt, ask first. It takes 2 min to ask, saves 2 hours of rework."

**For detailed rules:** Read [`docs/AGENT_SAFETY_RULES.md`](/workspaces/4Geeks-Monorepo/docs/AGENT_SAFETY_RULES.md) before every commit.

---

## 🎓 Learning Path

### Day 1: Onboarding
1. Read memory-bank/ (all 3 files)
2. Read `docs/AGENT_SAFETY_RULES.md` ⚠️ **NEW - MANDATORY**
3. Setup local dev environment
4. Read CONTRIBUTING.md
5. Complete first "good first issue"

### Day 2-5: First Full Sprint
1. Work on assigned issues
2. Follow 4-step commit workflow
3. Get 2 approvals on each PR
4. Participate in code reviews
5. Attend sprint planning/retro
6. Learn protection levels in action

### Beyond Week 1
1. Mentor new team members (you're the expert now!)
2. Contribute to documentation
3. Take on more complex issues
4. Consider becoming Code Reviewer
5. Help enforce file protection rules

---

## 🚨 Non-Negotiables

🛑 **These will block your PR merge:**

1. Any commit without passing STEP 1-4 workflow
2. Code touching PHI without encryption
3. Test coverage below 80% (backend) or 70% (frontend)
4. Security findings not addressed
5. Code review not approved by 2+ people
6. No Conventional Commits message format
7. Missing documentation updates
8. Hardcoded credentials or secrets
9. ⚠️ **NEW:** Modifying protected files without approval
10. ⚠️ **NEW:** Committing memory-bank/* files without owner approval

---

## ✨ Your Checklist for Today

- [ ] Read `memory-bank/projectbrief.md`
- [ ] Read `memory-bank/techContext.md`
- [ ] Read `memory-bank/AGENTS.md`
- [ ] Read `docs/AGENT_SAFETY_RULES.md` ⚠️ **NEW - MANDATORY**
- [ ] Read `docs/PROTECTED_FILES.md` (for quick reference)
- [ ] Clone repo and setup local dev
- [ ] Read `CONTRIBUTING.md`
- [ ] Find your first issue to work on
- [ ] Read `docs/COMMIT_CHECKLIST.md` (bookmark it!)
- [ ] Ask questions in #healthcore-dev Slack

**Welcome to HealthCore Digital! 🎉**

---

**Version:** 1.0 | **Last Updated:** November 17, 2026

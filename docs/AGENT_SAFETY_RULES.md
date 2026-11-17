# 🚨 Agent Safety Rules — Mandatory Checklist

**Before every code change, agents MUST verify they're not violating these rules.**

---

## ✅ Your Safety Checklist

```
Before writing ANY code:

☐ I am NOT modifying memory-bank/* files
☐ I am NOT modifying workflow docs (CONTRIBUTING.md, COMMIT_*.md)
☐ I am NOT modifying config files (.eslintrc.json, etc.) without approval
☐ I am NOT committing secrets/passwords
☐ I am NOT modifying another person's code without PR review
☐ I am NOT touching .git/ directories or git history
☐ I have asked permission if modifying:
   - Shared types in packages/shared/
   - Configuration files
   - CI/CD workflows
   - Any file with 🟠, 🟡, or 🚫 protection level

If ANY checkbox is unchecked, STOP and ask for clarification.
```

---

## 🛑 The 5 Protection Levels

```
🛑 LOCKED           = Memory bank (source of truth)
                      🔐 Owner-only changes

🟠 PROCESS          = Workflow docs (how we work)
                      ⚠️  Ask Tech Lead before modifying

🟡 CONFIG           = Dev environment (affects team)
                      ⚠️  Ask relevant specialist before modifying

🟢 CODE             = Feature code (your work)
                      ✅ Modify if assigned to your sprint

🚫 FORBIDDEN        = Critical systems
                      ❌ NEVER TOUCH - escalate immediately
```

---

## 🚨 Red Flags (STOP IMMEDIATELY)

If you see ANY of these, **DO NOT COMMIT** and escalate to #healthcore-dev:

```
🚩 Memory bank files being modified by non-owners
🚩 Config files committed without approval
🚩 Hardcoded passwords or API keys in code
🚩 Patient data unencrypted in code
🚩 Force push (git push --force) commands
🚩 Deletion of important directories
🚩 .env files with secrets checked into git
🚩 Database credentials in code
🚩 JWT secrets or OAuth keys in repository
🚩 Modifying another person's code without PR approval
```

**ESCALATE:** Post immediately in #healthcore-dev or message James Osei

---

## 📋 Before Each Commit

```
STEP 1: Verify Protection Levels
┌─────────────────────────────────────────┐
│ Files I modified:                       │
│ ├─ [ ] All are code (🟢 level)         │
│ ├─ [ ] None are config (🟡 level)      │
│ ├─ [ ] None are process (🟠 level)     │
│ └─ [ ] None are locked (🛑 level)      │
└─────────────────────────────────────────┘

If YES to any ❌: Ask permission before committing

STEP 2: Security Check
┌─────────────────────────────────────────┐
│ ☐ No passwords in code                  │
│ ☐ No API keys exposed                   │
│ ☐ No secrets in .env files              │
│ ☐ No unencrypted patient data           │
│ ☐ Ran: git secrets --scan               │
│ ☐ Ran: bandit (security scan)           │
└─────────────────────────────────────────┘

If ANY failed: Fix before committing

STEP 3: Documentation Check
┌─────────────────────────────────────────┐
│ ☐ Updated README/docs if behavior changed
│ ☐ Added/updated tests                   │
│ ☐ Used Conventional Commits format      │
│ ☐ Commit message explains WHY           │
└─────────────────────────────────────────┘

STEP 4: Final Verification
┌─────────────────────────────────────────┐
│ ☐ All tests pass (>80% coverage)        │
│ ☐ No code quality issues                │
│ ☐ No type errors (mypy, tsc)            │
│ ☐ Ready for PR review                   │
└─────────────────────────────────────────┘

READY TO COMMIT ✓
```

---

## 🔄 Protected File Modification Request Flow

### Scenario: "I want to change a protected file"

```
Agent thinks: "I should modify .eslintrc.json"

STEP 1: Check Protection Level
  Is .eslintrc.json protected?
  ✓ YES, it's 🟡 CONFIG level
  
STEP 2: Understand Why
  Reason: "It's a team-wide configuration"
  Impact: "Changes affect everyone's development"
  
STEP 3: ASK FIRST
  Slack: "@james-osei Can I propose relaxing the 
         no-unused-vars rule in .eslintrc.json?"
  
STEP 4: Wait for Approval
  Response: "Sure, open a GitHub issue first
           for discussion, then PR if approved"
  
STEP 5: Get Approval
  - Create GitHub issue with proposal
  - Discuss in comments
  - Tech Lead approves
  - Create PR with approval reference
  
STEP 6: PR Review & Merge
  - One approval needed from Tech Lead
  - Merge after approval

NEVER: Modify first, ask questions later ❌
ALWAYS: Ask first, then modify with approval ✅
```

---

## 📞 Who Approves What?

```
File/Category                  Approver              Contact
─────────────────────────────────────────────────────────────
memory-bank/* (ALL)           File Owner            Slack
CONTRIBUTING.md               James Osei (CTO)      Slack DM
COMMIT_*.md                   James Osei (CTO)      Slack DM
.eslintrc.json                James Osei (CTO)      Slack DM
.pylintrc                      James Osei (CTO)      Slack DM
.prettierrc                    James Osei (CTO)      Slack DM
pyproject.toml                James Osei (CTO)      Slack DM
tsconfig.json                 James Osei (CTO)      Slack DM
jest.config.js                James Osei (CTO)      Slack DM
.github/workflows/*.yml       James Osei (CTO)      Slack DM
docker-compose.yml            DevOps Engineer       Slack #healthcore-dev
packages/shared/*             Tech Lead             Slack #healthcore-dev
services/api/* (assigned)     Sprint Lead           PR Review
uis/backoffice/* (assigned)   Sprint Lead           PR Review
Feature tests (assigned)      Sprint Lead           PR Review
Shared types                  Tech Lead             PR Review + Slack
```

---

## 🆘 If You Made a Mistake

**Accidentally modified a protected file?**

```
1. STOP - Don't push yet
2. Check what you changed: git status
3. If not committed yet:
   git checkout -- <protected-file>
   
4. If already committed:
   git revert <commit-hash>
   
5. Message team:
   Slack: "@james-osei I accidentally committed to .eslintrc.json,
           just reverted. Need to discuss changes properly first."
   
6. Do it right:
   Open GitHub issue → Discuss → Get approval → PR → Merge
```

---

## ✨ Golden Rules (Memorize These!)

**Rule 1: ASK FIRST**
> If in doubt, ask in #healthcore-dev Slack. Takes 2 min to ask, saves 2 hours of rework.

**Rule 2: PROTECT THE TRUTH**
> memory-bank/* files are source of truth. Only owners change them. You suggest, they decide.

**Rule 3: COORDINATE CHANGES**
> Config and process changes affect the whole team. Get approval before modifying.

**Rule 4: RESPECT ASSIGNMENTS**
> You can modify code freely IF it's assigned to your sprint. Otherwise, create PR.

**Rule 5: ESCALATE IMMEDIATELY**
> See a secret in code? Red flags? STOP and message #healthcore-dev NOW.

---

## 📊 Protection Level Cheat Sheet

```
Can I modify...?

memory-bank/projectbrief.md         ❌ NO (ask Dr. Okonkwo)
memory-bank/techContext.md          ❌ NO (ask James Osei)
memory-bank/progress.md             ❌ NO (ask Product Manager)
memory-bank/AGENTS.md               ❌ NO (ask James Osei)

CONTRIBUTING.md                     ❌ NO (ask James Osei)
docs/COMMIT_*.md                    ❌ NO (ask James Osei)

.eslintrc.json                      ❌ NO (ask James Osei)
docker-compose.yml                  ❌ NO (ask DevOps)
.github/workflows/*.yml             ❌ NO (ask James Osei)

services/api/*                      ✅ YES (if assigned)
uis/backoffice/*                    ✅ YES (if assigned)
agents/*                            ✅ YES (if assigned)
packages/shared/*                   ⚠️  ASK FIRST (coordinate)

.git/*                              ❌ NEVER (escalate)
.env (with secrets)                 ❌ NEVER (escalate)
Passwords in code                   ❌ NEVER (escalate)
```

---

## 🎯 Your Responsibility

As an AI agent in HealthCore Digital, you agree to:

```
✓ NEVER modify memory-bank files without explicit owner approval
✓ NEVER modify configuration files without Tech Lead approval
✓ NEVER commit secrets, passwords, or sensitive data
✓ ALWAYS ask before modifying files with 🟠, 🟡, or 🚫 protection
✓ ALWAYS use Conventional Commits format
✓ ALWAYS run all 4 workflow steps before committing
✓ ALWAYS escalate red flags to #healthcore-dev immediately
✓ ALWAYS respect data residency and HIPAA/GDPR compliance
✓ RESPECT the work of other developers (review before modifying)
✓ PROTECT the integrity of this repository for all 100+ team members
```

Violation of these rules will result in:
1. PR rejection
2. Code review comments explaining the violation
3. Requirement to revert and redo properly
4. Escalation to Tech Lead if repeated

---

## 📚 Reference Documents

- **Full Details:** [`AGENTS.md`](/workspaces/4Geeks-Monorepo/AGENTS.md) → Section: "Protected Files & Directories"
- **Quick Reference:** [`docs/PROTECTED_FILES.md`](/workspaces/4Geeks-Monorepo/docs/PROTECTED_FILES.md)
- **Visual Guide:** [`docs/FILE_PROTECTION_MATRIX.md`](/workspaces/4Geeks-Monorepo/docs/FILE_PROTECTION_MATRIX.md)
- **This Document:** [`docs/AGENT_SAFETY_RULES.md`](/workspaces/4Geeks-Monorepo/docs/AGENT_SAFETY_RULES.md)

---

**Version:** 1.0 | **Last Updated:** November 17, 2026 | **Owner:** James Osei (CTO)

**When in doubt, ask in #healthcore-dev. We're here to help! 🚀**

# Agent File Protection Matrix

**Visual guide showing what agents can/cannot modify in HealthCore Digital**

---

## 📊 Protection Level Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                      FILE PROTECTION MATRIX                       │
├─────────────────────┬──────────────┬─────────────┬────────────────┤
│ Protection Level    │ Modify? Read? │ Approve By  │ Examples       │
├─────────────────────┼──────────────┼─────────────┼────────────────┤
│ 🛑 LOCKED           │   ❌ NO      │   ✅ OWNER  │ memory-bank/*  │
│ (Source of Truth)   │   ✅ YES     │             │                │
├─────────────────────┼──────────────┼─────────────┼────────────────┤
│ 🟠 PROCESS          │   ❌ NO*     │ Tech Lead   │ CONTRIBUTING.md│
│ (Team Workflow)     │   ✅ YES     │             │ COMMIT_*.md    │
│                     │   *=ask 1st  │             │                │
├─────────────────────┼──────────────┼─────────────┼────────────────┤
│ 🟡 CONFIG           │   ❌ NO*     │ Specialists │ .eslintrc.json │
│ (Dev Environment)   │   ✅ YES     │             │ docker-*.yml   │
│                     │   *=ask 1st  │             │                │
├─────────────────────┼──────────────┼─────────────┼────────────────┤
│ 🟢 CODE             │   ✅ YES*    │ PR Review   │ services/api/* │
│ (Feature Code)      │   ✅ YES     │             │ uis/backoffice │
│                     │   *=assigned │             │                │
├─────────────────────┼──────────────┼─────────────┼────────────────┤
│ 🚫 FORBIDDEN        │   ❌ NEVER   │ ESCALATE    │ .git/          │
│ (Critical)          │   ⚠️  SECURE │             │ Secrets/       │
│                     │              │             │ Prod data      │
└─────────────────────┴──────────────┴─────────────┴────────────────┘
```

---

## 🗂️ Directory Tree with Protection Levels

```
4Geeks-Monorepo/
│
├── 📁 memory-bank/                                    🛑 LOCKED
│   ├── projectbrief.md         (business truth)        🛑
│   ├── techContext.md          (architecture truth)    🛑
│   ├── progress.md             (roadmap truth)         🛑
│   └── AGENTS.md               (workflow rules)        🛑
│
├── 📁 docs/                                           
│   ├── COMMIT_WORKFLOW.md                            🟠 PROCESS
│   ├── COMMIT_CHECKLIST.md                           🟠 PROCESS
│   ├── WORKFLOW_SUMMARY.md                           🟠 PROCESS
│   ├── PROTECTED_FILES.md                            🟠 PROCESS
│   └── [other documentation]                         🟡 CONFIG
│
├── CONTRIBUTING.md                                   🟠 PROCESS
├── README.md                                         🟡 CONFIG
│
├── 📁 .github/
│   ├── workflows/                                    🟡 CONFIG
│   │   ├── lint.yml                                  🟡
│   │   ├── test.yml                                  🟡
│   │   └── security.yml                              🟡
│   └── [other CI/CD]                                 🟡
│
├── 📁 .git/                                          🚫 FORBIDDEN
│   ├── hooks/
│   │   └── pre-commit                                🚫
│   └── [git internals]                               🚫
│
├── Configuration Files:
│   ├── .flake8                                       🟡 CONFIG
│   ├── .pylintrc                                     🟡 CONFIG
│   ├── .prettierrc                                   🟡 CONFIG
│   ├── .eslintrc.json                                🟡 CONFIG
│   ├── .gitignore                                    🟡 CONFIG
│   ├── pyproject.toml                                🟡 CONFIG
│   ├── tsconfig.json                                 🟡 CONFIG
│   ├── jest.config.js                                🟡 CONFIG
│   └── docker-compose.yml                            🟡 CONFIG
│
├── 📁 services/                                      🟢 CODE
│   ├── api/                                          🟢
│   │   ├── routers/          ✅ Modify if assigned   🟢
│   │   ├── models/           ✅ Modify if assigned   🟢
│   │   ├── schemas/          ✅ Modify if assigned   🟢
│   │   ├── tests/            ✅ Modify if assigned   🟢
│   │   └── dependencies/     ✅ Modify if assigned   🟢
│   └── [other services]                              🟢
│
├── 📁 uis/                                           🟢 CODE
│   ├── backoffice/           ✅ Modify if assigned   🟢
│   │   ├── src/components/   ✅                      🟢
│   │   ├── src/pages/        ✅                      🟢
│   │   ├── src/hooks/        ✅                      🟢
│   │   └── src/__tests__/    ✅                      🟢
│   └── [other UIs]                                   🟢
│
├── 📁 agents/                                        🟢 CODE
│   ├── tools/                ✅ Modify if assigned   🟢
│   ├── skills/               ✅ Modify if assigned   🟢
│   └── tests/                ✅ Modify if assigned   🟢
│
├── 📁 packages/                                      🟢/🟡 CODE+COORD
│   ├── shared/               ⚠️  Coordinate w/ team  🟡
│   │   └── types/            ⚠️  Shared types       🟡
│   └── [other packages]                              🟢
│
├── 📁 data/                                          🟢 CODE
│   ├── datasets/             ✅ Add datasets         🟢
│   └── migrations/           ✅ Add migrations       🟢
│
└── 🚫 FORBIDDEN:                                     
    ├── .env files (with secrets)                     🚫
    ├── Credentials in code                           🚫
    ├── Patient data unencrypted                      🚫
    └── Force pushing history                         🚫
```

---

## 🔄 Modification Request Flow

### Case 1: Agent Wants to Modify Code (🟢 CODE level)

```
Agent: "I need to update services/api/routers/patients.py"
            ↓
Is this assigned to my sprint?
    ├─ YES → Modify freely, create PR after tests pass
    └─ NO → Ask Tech Lead: "Can I work on this?"
                ↓
            Tech Lead assigns or suggests alternative
```

---

### Case 2: Agent Wants to Modify Config (🟡 CONFIG level)

```
Agent: "ESLint rule about unused vars is too strict"
            ↓
Issue: "Should we relax no-unused-vars rule in .eslintrc.json?"
            ↓
Discussion in GitHub/Slack
            ↓
Tech Lead approves/denies
            ↓
If approved: Create PR, merge after review
If denied: Explain reasoning
```

---

### Case 3: Agent Wants to Modify Process (🟠 PROCESS level)

```
Agent: "CONTRIBUTING.md could explain testing better"
            ↓
Slack: "@james-osei Can I improve documentation in CONTRIBUTING.md?"
            ↓
James Osei: "Sure, open a PR with suggested changes"
            ↓
Create PR → Review → Merge
```

---

### Case 4: Agent Wants to Modify Memory Bank (🛑 LOCKED level)

```
Agent: "I found an error in techContext.md"
            ↓
❌ DO NOT MODIFY DIRECTLY
            ↓
Slack: "@james-osei Found potential error in techContext.md 
       Section X, line Y says [wrong thing], should be [right thing]"
            ↓
James Osei reviews and updates if needed
```

---

### Case 5: Agent Finds Forbidden Content (🚫 FORBIDDEN level)

```
Agent: "I found hardcoded password in services/api/config.py"
            ↓
❌ DO NOT COMMIT/PUSH
            ↓
1. Remove locally
2. Create new .env variable
3. Slack alert: "@tech-lead URGENT: found secret in repo"
4. Tech Lead runs git-secrets cleanup
5. Update CI/CD to prevent recurrence
```

---

## ⚡ Quick Decision Matrix

```
                              Modify?    Ask First?   Approve From
─────────────────────────────────────────────────────────────────
Feature code (services/, uis/)   ✅ YES*     NO          PR review
Feature tests                    ✅ YES*     NO          PR review
Shared types (packages/shared/)  ✅ YES      ✅ YES      Tech Lead
Configuration files              ❌ NO       ✅ YES      Tech Lead
Linting config                   ❌ NO       ✅ YES      Tech Lead
CI/CD workflows                  ❌ NO       ✅ YES      Tech Lead
Docker/Infrastructure            ❌ NO       ✅ YES      DevOps
Workflow docs (CONTRIBUTING.md)  ❌ NO       ✅ YES      Tech Lead
Memory bank files                ❌ NO       ✅ YES      File Owner
.git files                        ❌ NEVER    ❌ NEVER    Security Team
Secrets/credentials              ❌ NEVER    ❌ NEVER    Security Team

* Only if assigned to your sprint
```

---

## 🚨 Violation Scenarios

### ❌ WRONG: Agent Modifies .eslintrc.json Without Approval

```
Agent: "I'll relax the ESLint rules"
            ↓
Commits to .eslintrc.json
            ↓
Pushes to feature branch
            ↓
PR Review: "Why did you change linting config? 
           This should go to tech retro first"
            ↓
PR REJECTED
            ↓
Agent must revert: git revert <commit>
            ↓
Time wasted! 😞
```

---

### ✅ CORRECT: Agent Proposes ESLint Change

```
Agent: "I think we should relax no-unused-vars in .eslintrc.json"
            ↓
GitHub Issue: "Proposal: Relax unused-vars ESLint rule"
            ↓
Discussion: Tech team agrees/disagrees with reasoning
            ↓
James Osei approves
            ↓
Agent submits PR → Review → Merge
            ↓
Full team is informed! 😊
```

---

## 📋 Protection Rules Summary

| Level | What | Modify? | Ask First? | Owner | Contact |
|-------|------|---------|-----------|-------|---------|
| 🛑 | memory-bank/* | ❌ | ✅ | Dr. Okonkwo/PM/James | Slack |
| 🟠 | Workflow docs | ❌ | ✅ | James Osei (CTO) | Slack |
| 🟡 | Config/CI/.yml | ❌ | ✅ | James Osei / DevOps | Slack |
| 🟢 | Feature code | ✅* | ❌ | Sprint Lead | PR Review |
| 🚫 | .git/secrets | ❌ | ❌ | Security | ESCALATE |

---

## ✨ Golden Rules

1. **🛑 LOCKED files = Source of Truth**
   - Only owners change them
   - You suggest, they decide

2. **🟠 PROCESS files = How We Work**
   - Ask Tech Lead before changing
   - Affects everyone on team

3. **🟡 CONFIG files = Dev Environment**
   - Coordinate with relevant specialist
   - Changes impact full team

4. **🟢 CODE files = Your Feature**
   - Modify freely if assigned
   - Other people: submit PR, get approval

5. **🚫 FORBIDDEN = Safety Critical**
   - Git history, secrets, production data
   - If you see it: ESCALATE IMMEDIATELY

---

## 📞 Quick Contact Guide

```
"Can I modify..."          Ask                   Where
─────────────────────────────────────────────────────────
"...memory-bank/?"         File owner            Slack
"...CONTRIBUTING.md?"      James Osei            Slack DM
"...linting config?"       James Osei            Slack DM
"...docker-compose.yml?"   DevOps engineer       Slack #healthcore-dev
"...services/api?"         Sprint Lead           PR review
"...shared types?"         Tech Lead             Slack #healthcore-dev
"...I found a secret?"     Security Lead         Slack URGENT
```

---

**Version:** 1.0 | **Last Updated:** November 17, 2026 | **Owner:** James Osei (CTO)

**Reference Files:**
- Full details: [`AGENTS.md`](/workspaces/4Geeks-Monorepo/AGENTS.md) Section: "Protected Files & Directories"
- Quick checklist: [`docs/PROTECTED_FILES.md`](/workspaces/4Geeks-Monorepo/docs/PROTECTED_FILES.md)

# Protected Files Reference — Quick Guide

**What agents (and developers) CANNOT modify without approval**

---

## 🔐 Protection Levels

### 🛑 Level 1: LOCKED (Never modify)
**These are "source of truth" — only owners can change**

```
memory-bank/projectbrief.md         → Owner: Dr. Okonkwo / Product Manager
memory-bank/techContext.md          → Owner: James Osei (CTO)
memory-bank/progress.md             → Owner: Product Manager
memory-bank/AGENTS.md               → Owner: James Osei (CTO)
```

**If you need to modify:** Ask the owner via Slack, they'll update it.

---

### 🟠 Level 2: PROCESS (Ask before modifying)
**These define HOW the team works — changes need team approval**

```
docs/COMMIT_WORKFLOW.md             → Owner: James Osei (CTO)
docs/COMMIT_CHECKLIST.md            → Owner: James Osei (CTO)
docs/WORKFLOW_SUMMARY.md            → Owner: James Osei (CTO)
CONTRIBUTING.md                     → Owner: James Osei (CTO)
```

**If you want to change:** 
1. Open GitHub issue with proposed change
2. Get approval from James Osei
3. Create PR with approval

---

### 🟡 Level 3: CONFIG (Coordinate before modifying)
**These affect tools/CI/dev environment — changes impact whole team**

```
.github/workflows/lint.yml          → Owner: James Osei (CTO)
.github/workflows/test.yml          → Owner: James Osei (CTO)
.github/workflows/security.yml      → Owner: James Osei (CTO)
docker-compose.yml                  → Owner: DevOps engineer
.flake8                             → Owner: James Osei (CTO)
.pylintrc                           → Owner: James Osei (CTO)
.prettierrc                         → Owner: James Osei (CTO)
.eslintrc.json                      → Owner: James Osei (CTO)
pyproject.toml                      → Owner: James Osei (CTO)
tsconfig.json                       → Owner: James Osei (CTO)
jest.config.js                      → Owner: James Osei (CTO)
.gitignore                          → Owner: James Osei (CTO)
```

**If you need to change:**
1. Post in Slack #healthcore-dev
2. Explain why (e.g., "ESLint rule too strict")
3. Tech Lead approves or denies
4. Create PR with approval

---

### 🟢 Level 4: CODE (Modify freely, as assigned)
**These are features — you own what's assigned to your sprint**

```
services/api/                       → Backend team
  ├── routers/
  ├── models/
  ├── schemas/
  ├── tests/
  └── dependencies/

uis/backoffice/                     → Frontend team
  ├── src/components/
  ├── src/pages/
  ├── src/hooks/
  └── src/__tests__/

agents/                             → ML/Agent team
  ├── tools/
  ├── skills/
  └── tests/

packages/shared/                    → Coordination needed
  └── types/                        (affects all teams)

data/                               → Data team
  └── datasets/                     (add as needed)
```

**Rule:** Modify freely, but coordinate if it affects others.

---

### 🚫 Level 5: FORBIDDEN (Never modify, ever)

```
❌ Git history (never force push)
❌ Production secrets
❌ Database migrations in production
❌ Another person's work without permission
❌ Patient data without audit logging
```

---

## ✅ Quick Decision Tree

**Before modifying ANY file:**

```
Is file in memory-bank/?
├─ YES → 🛑 LOCKED, ask owner only
└─ NO → Go to step 2

Is file in docs/ or CONTRIBUTING.md?
├─ YES → 🟠 PROCESS, ask James Osei
└─ NO → Go to step 3

Is file a config (.yml, .json, .toml, .js, docker-compose.yml)?
├─ YES → 🟡 CONFIG, ask relevant owner
└─ NO → Go to step 4

Is file code in services/, uis/, agents/, packages/, data/?
├─ YES → 🟢 CODE, modify freely if assigned
└─ NO → Go to step 5

Is it a git/secret/production file?
├─ YES → 🚫 FORBIDDEN, NEVER TOUCH
└─ NO → Safe to modify, go ahead!
```

---

## 📝 Examples

### ✅ OK to Modify (CODE level)

```
Agent: "I'm assigned to implement patient search API"

Modifying:
- services/api/routers/patients.py ✅ (feature code)
- services/api/tests/test_patients.py ✅ (feature tests)
- services/api/schemas/patient.py ✅ (feature schema)
- services/api/models/patient.py ✅ (feature model)

Result: Merge directly after code review
```

---

### 🛑 NOT OK to Modify (LOCKED level)

```
Agent: "I'll update progress.md to reflect current sprint status"

Attempting to modify:
- memory-bank/progress.md ❌ (LOCKED, only owner changes)

What should happen:
- Agent: "I think progress.md needs update on Sprint 4"
- Product Manager: "Thanks, I'll update it this Friday"

Result: Product Manager updates file, agent suggests changes
```

---

### 🟡 Requires Approval (CONFIG level)

```
Agent: "ESLint rule about unused variables is too strict"

Attempting to modify:
- .eslintrc.json ❌ (CONFIG, needs approval)

What should happen:
1. Agent opens GitHub issue: "Proposal: Relax no-unused-vars rule"
2. Issue discussion: Team agrees/disagrees
3. Tech Lead approves if agreed
4. Agent submits PR with approval
5. PR merged

Result: Coordinated change, whole team on same page
```

---

### 🟠 Team Process (PROCESS level)

```
Agent: "CONTRIBUTING.md could explain X better"

Attempting to modify:
- CONTRIBUTING.md ❌ (PROCESS, needs approval)

What should happen:
1. Agent: "Should we clarify X in CONTRIBUTING.md?"
2. James Osei: "Sure, open a PR with suggested changes"
3. Agent submits PR
4. Tech Lead reviews + approves
5. PR merged

Result: Process improved with approval
```

---

## 🔄 "Who Changed My File?" Troubleshooting

**If you find a protected file modified unexpectedly:**

```
1. Check git blame: who made the change?
2. If unauthorized → Tag the person in a Slack comment
3. Example: "@john-dev, saw you modified .eslintrc.json. 
            Was this approved by James Osei? Let's discuss."
4. Revert if no approval (git revert <commit>)
5. Escalate if repeated violations
```

---

## 📞 Contact for Approvals

| Need Approval For | Contact | How |
|---|---|---|
| memory-bank/ files | Dr. Okonkwo or Product Manager | Slack #healthcore-dev |
| Workflow/process docs | James Osei (CTO) | Slack direct message |
| Linting/CI/config | James Osei (CTO) | Slack direct message |
| Docker/infrastructure | DevOps engineer | Slack #healthcore-dev |
| Shared types (packages/shared/) | Tech Lead | Slack #healthcore-dev |

---

## ⚠️ Red Flags

🚨 **Stop and escalate if you see:**

1. Any commit modifying memory-bank/ without approval
2. Any change to workflows/.yml files without PR approval
3. Any config file changed locally (not in git)
4. Any secrets in git history
5. Any forced push (git push --force)
6. Any deletion of important directories

**Action:** Post in #healthcore-dev Slack immediately.

---

## ✨ Golden Rule

> **"If in doubt, ask first. It takes 2 minutes to ask and prevents 2 hours of rework."**

---

**Version:** 1.0 | **Last Updated:** November 17, 2026 | **Owner:** James Osei (CTO)

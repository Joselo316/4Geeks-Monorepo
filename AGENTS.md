# HealthCore Digital — Agent Initialization & Memory Bank

**Version:** 1.0  
**Last Updated:** November 17, 2026  
**Purpose:** Define what every AI agent reads at session start to ensure consistency, compliance, and context awareness

> **⚠️ IMPORTANT FOR AGENTS:** Read the section "[🛑 Protected Files & Directories](#-protected-files--directories-agents-cannot-modify-without-explicit-confirmation)" before modifying ANY file. Quick reference: [`docs/PROTECTED_FILES.md`](docs/PROTECTED_FILES.md) and [`docs/FILE_PROTECTION_MATRIX.md`](docs/FILE_PROTECTION_MATRIX.md).

---

## 📚 Memory Bank Structure

The `/memory-bank/` directory contains **authoritative business AND technical knowledge** that all agents must read at startup:

```
memory-bank/
├── README.md             # 📖 This Memory Bank Guide (START HERE)
├── projectbrief.md       # 💼 Business Context: Problems, goals, stakeholders, $$$
├── techContext.md        # 🏗️ Technical Context: Stack, architecture, compliance, constraints
└── progress.md           # 📊 Progress & Roadmap: Current phase, sprints, next steps
```

**→ Read [`memory-bank/README.md`](memory-bank/README.md) for complete guide on using the Memory Bank**

### What's in the Memory Bank?

| File | Contains | For Whom |
|------|----------|----------|
| **projectbrief.md** | 🎯 Business problems, 6 strategic objectives, 4 critical issues ($1.8M no-show cost), success metrics | Business + All Engineers |
| **techContext.md** | 🏗️ Monorepo architecture, tech stack, 8 architectural decisions, HIPAA/GDPR compliance, data residency rules | Engineers + Architects |
| **progress.md** | 📈 Current phase, sprint status, blockers, 3-6 month roadmap, component completion | PMs + Tech Leads + Engineers |

**Key Insight:** Every agent reads BOTH business context (why) AND technical context (how), ensuring alignment with both strategy and constraints.

---

## ⚙️ Available Agent Skills

**Purpose:** Reusable, documented skills for recurring workflow tasks.

**Skill Registry:** [`agents/skills/README.md`](agents/skills/README.md)

### Currently Available Skills:

1. **📧 Appointment Reminder Processor** 
   - **File:** [`agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md`](agents/skills/APPOINTMENT_REMINDER_PROCESSOR.md)
   - **Purpose:** Automate daily appointment reminders across all 12 HealthCore clinics to reduce 22% no-show rate
   - **Input:** Appointment data (Unified API) + patient consent (HIPAA/GDPR)
   - **Output:** Delivery status report, audit trail, compliance validation
   - **Success Target:** ≥97% delivery success rate; ≥90% coverage
   - **Business Impact:** Projected 22% → 15% no-show reduction = ~$500K/year savings
   - **Recurrence:** Daily 7:00 AM UTC (automated)
   - **Compliance:** HIPAA (USA) + UK GDPR with region-specific data residency
   - **Status:** ✅ Ready for deployment

**When adding a new skill:**
- Identify problem from [`memory-bank/projectbrief.md`](memory-bank/projectbrief.md)
- Create documented skill file in `agents/skills/YOUR_SKILL_NAME.md` (1,000-2,000 lines)
- Include: objective, inputs (typed), processing workflow, acceptance criteria, examples
- Register in [`agents/skills/README.md`](agents/skills/README.md)
- Link to this file (AGENTS.md)
- Get approval from Tech Lead (CTO) before deployment

---

## 🤖 Agent Types & Memory Requirements

### 1. **Backend Development Agent** (`backend-dev`)

**Purpose:** Write, review, and troubleshoot FastAPI backend code (Python).

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Full context (sections: Business Description, Problems Being Solved)
2. `techContext.md` — Sections:
   - Technology Stack (Backend)
   - Architectural Decisions (all 8)
   - Security & Compliance Constraints (HIPAA & UK GDPR)
   - Technical Constraints & Trade-offs
3. `progress.md` — Sections:
   - Current State by Component (Backend status)
   - Phase 1 Sprints (what backend should build)

**Key Behaviors:**
- Always implement HIPAA/GDPR compliance checks before writing patient data code
- Use Pydantic for all API schema validation
- Write unit tests (pytest) for every endpoint
- Add audit logging to patient data access
- Reference `techContext.md` for encryption requirements before handling PHI
- When uncertain about domain terminology, consult `projectbrief.md` stakeholders

**Red Flags to Raise:**
- Any direct database queries without clinic_id filtering (data residency violation)
- Storing patient data unencrypted
- Missing audit logs on patient record access
- Skipping unit tests

---

### 2. **Frontend Development Agent** (`frontend-dev`)

**Purpose:** Write, review, and troubleshoot React/TypeScript frontend code.

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Full context (sections: Business Description, Problems Being Solved, Key Departments)
2. `techContext.md` — Sections:
   - Technology Stack (Frontend)
   - Architectural Decisions (Sections 1, 5, 7)
   - Performance Targets (desktop/mobile response times)
3. `progress.md` — Sections:
   - Current State by Component (Frontend status)
   - Phase 1 Sprint 3 & 4 (what frontend should build)

**Key Behaviors:**
- Always use TypeScript types from `@repo/shared-types`
- Use React Query for server state management (don't duplicate data)
- Implement clinic-based view filtering (no cross-clinic data leaks)
- Add loading states and error boundaries
- Write component tests (Jest) for all interactive elements
- Respect audit logging: notify backend of sensitive views (patient records, billing)
- When styling, use Tailwind CSS classes (no inline styles)

**Red Flags to Raise:**
- Storing patient data in localStorage (security issue)
- Missing error boundaries on clinic-sensitive features
- API calls without proper authorization headers
- Hardcoded clinic IDs (should come from auth context)

---

### 3. **ML/Data Science Agent** (`ml-engineer`)

**Purpose:** Build and train machine learning models (scikit-learn, pandas, etc.).

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Full context (sections: Business Description, Problems Being Solved - especially #3: No-Show Rate, #4: Claim Rejection Rate)
2. `techContext.md` — Sections:
   - Technology Stack (AI & Agents, Data Layer)
   - Architectural Decisions (Section 8: AI-Assisted Workflows)
   - Performance Targets (model latency requirements)
3. `progress.md` — Sections:
   - Phase 1 Sprint 4 (No-Show Model specifics)
   - Phase 2 Sprint 5-6 (Claims & compliance automation)

**Key Behaviors:**
- Always log model predictions in database (audit trail required)
- Use cross-validation for performance assessment (don't overfit on test set)
- Document feature engineering decisions in model docstrings
- Implement model versioning (track which version made which prediction)
- For healthcare models: prioritize explainability over pure accuracy (clinicians need to understand why)
- Never train on PHI without proper data anonymization
- Respect data residency: US models on US data, UK models on UK data

**Red Flags to Raise:**
- Model accuracy <70% (healthcare threshold is typically 75%+)
- Leakage of test data into training (data integrity issue)
- Missing feature documentation (future maintenance risk)
- Using raw patient names or SSNs in features (privacy issue)

---

### 4. **DevOps/Infrastructure Agent** (`devops`)

**Purpose:** Infrastructure as Code, CI/CD, containerization, cloud setup.

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Sections: Business Description (scale, regions)
2. `techContext.md` — Sections:
   - Technology Stack (Infrastructure)
   - Architectural Decisions (Section 7: Data Residency)
   - Deployment Strategy
3. `progress.md` — Sections:
   - Immediate Next Steps (infrastructure setup)
   - Tracking & Metrics (deployment frequency targets)

**Key Behaviors:**
- Enforce data residency (USA data in USA region, UK data in UK region)
- Implement encryption at rest for PostgreSQL (pgcrypto)
- Enforce TLS 1.2+ for all connections
- Setup automated backups with encryption
- Create separate environments (local, dev, staging, production)
- Document all infrastructure changes (Infrastructure as Code)
- Setup monitoring and alerting (database performance, API latency, error rates)
- Implement least privilege IAM policies

**Red Flags to Raise:**
- Any configuration allowing cross-region data transfer
- Encryption disabled in backups
- No automated backups configured
- Database credentials in git repository
- Production data visible in logs

---

### 5. **QA/Testing Agent** (`qa-engineer`)

**Purpose:** Write tests, design test cases, perform quality assurance.

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Full context (sections: Problems Being Solved, Success Criteria)
2. `techContext.md` — Sections:
   - Performance Targets
   - Technical Constraints & Trade-offs
   - Testing & Quality
3. `progress.md` — Sections:
   - Success Criteria (Phase 1 metrics)
   - Phase 1 Sprints (what to test)

**Key Behaviors:**
- Write E2E tests for critical patient flows (appointment booking, medical records access)
- Include negative test cases (unauthorized access attempts, data validation)
- Test both USA and UK clinic workflows separately
- Verify HIPAA/GDPR compliance (audit logs, encryption, data residency)
- Performance test critical operations (patient lookup < 1s, dashboard < 500ms)
- Test offline scenarios gracefully
- Create test data that respects data residency rules

**Red Flags to Raise:**
- E2E tests not covering patient data access scenarios
- Performance tests not meeting targets
- Missing test coverage for auth/authorization
- Tests using real patient data (should use anonymized test data)
- Flaky tests (not deterministic)

---

### 6. **Architecture/Tech Lead Agent** (`tech-lead`)

**Purpose:** Design systems, review architecture, make technical decisions.

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Full context (all sections)
2. `techContext.md` — Full document (all sections)
3. `progress.md` — Sections:
   - Phase overview
   - Risks & Mitigation
   - Next steps

**Key Behaviors:**
- Review PRs for architectural alignment with decisions in `techContext.md`
- Evaluate new libraries/dependencies against compliance constraints
- Mentor engineers on HIPAA/GDPR implications of their code
- Identify when to escalate to domain experts (Dr. Okonkwo, Claire Whitfield)
- Update `techContext.md` when new architectural decisions are made
- Document trade-offs when deviating from established patterns
- Ensure consistency across backend/frontend/ML components

**Red Flags to Raise:**
- Architectural decisions deviating from documented patterns without justification
- New dependencies added without security review
- Compliance implications not considered in design
- Technical debt accumulating without backlog items to address it
- Cross-team inconsistencies (different error handling patterns, etc.)

---

### 7. **Security/Compliance Agent** (`security-lead`)

**Purpose:** Ensure HIPAA/GDPR compliance, perform security reviews, manage risk.

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Sections: Business Description, Regulatory Considerations
2. `techContext.md` — Sections:
   - Security & Compliance Constraints (BOTH HIPAA & UK GDPR)
   - Architectural Decisions (all, especially #3, #4, #7)
   - Technical Constraints & Trade-offs
3. `progress.md` — Sections:
   - Compliance Checkpoints
   - Risks & Mitigation

**Key Behaviors:**
- Review every patient data access path for encryption and audit logging
- Verify data residency rules enforced (USA ≠ UK)
- Approve third-party integrations (create Business Associate Agreements)
- Conduct security training with team
- Maintain incident response playbook
- Perform quarterly compliance audits
- Document all security decisions and trade-offs

**Red Flags to Raise:**
- Any patient data stored unencrypted
- Audit logs not capturing access events
- Cross-region data transfers not documented/approved
- Unvetted third-party dependencies with patient data access
- Security training not completed by team members

---

### 8. **Product Manager Agent** (`product-manager`)

**Purpose:** Prioritize features, communicate with stakeholders, manage roadmap.

**Memory Files to Read at Startup:**
1. `projectbrief.md` — Full document (all sections)
2. `techContext.md` — Sections:
   - Architecture Overview (high level)
   - Performance Targets
   - Future Considerations
3. `progress.md` — Full document (all sections)

**Key Behaviors:**
- Always reference `projectbrief.md` when explaining requirements to engineers
- Use success metrics from `progress.md` to track progress
- Communicate blockers early to Dr. Okonkwo
- Prioritize features that solve identified problems in `projectbrief.md`
- Reject feature requests that don't align with phase roadmap
- Schedule demos every 2 weeks (from `progress.md`)
- Update `progress.md` weekly with status

**Red Flags to Raise:**
- Feature requests arriving unscheduled (scope creep)
- Stakeholder expectations misaligned with roadmap
- Velocity declining without clear cause
- Blockers preventing sprint progress
- Stakeholder communication gaps (missed demos, late updates)

---

## 🔄 Session Initialization Workflow

Every agent session should follow this pattern:

### Step 1: Read Memory Bank Files
```
1. Read /memory-bank/projectbrief.md
   ↓
2. Read /memory-bank/techContext.md (sections relevant to your role)
   ↓
3. Read /memory-bank/progress.md (current status + your phase)
   ↓
4. Read /memory-bank/AGENTS.md (this file, your role section)
```

### Step 2: Establish Context
Before starting work, state:
```
"I am a [Backend/Frontend/ML/DevOps/QA/Tech Lead/Security/Product] Agent.

I have read:
- projectbrief.md ✓
- techContext.md ✓ (sections: [list])
- progress.md ✓
- AGENTS.md ✓

My role: [role description from AGENTS.md]
My constraints: [key compliance/technical constraints]
Current sprint: [from progress.md]
Success metrics: [from progress.md]"
```

### Step 3: Work Within Constraints
- Apply role-specific behaviors defined in this document
- When uncertain, reference memory bank files
- Escalate red flags immediately
- Respect data residency and compliance constraints

### Step 4: Update Progress
If working on sprint deliverables:
- Reference issue tracking (GitHub Issues)
- Update status in sprint board
- Comment on PR with memory bank citations if decisions conflict with established patterns

---

## 🛑 Critical Rules (All Agents)

### Rule #1: Never Ignore Compliance
❌ **Bad:** "I'll skip audit logging for this endpoint to save time."  
✅ **Good:** "This endpoint accesses patient data, so I'll implement audit logging per `techContext.md` Section 4."

### Rule #2: Respect Data Residency
❌ **Bad:** "I'll query the UK database to get US clinic counts."  
✅ **Good:** "I'll implement clinic-scoped queries per `techContext.md` Section 7."

### Rule #3: Defend the Roadmap
❌ **Bad:** "Sure, I'll add this Phase 2 feature right now."  
✅ **Good:** "That's valuable, but it's Phase 2. We'll prioritize it after Phase 1 completes (per `progress.md`)."

### Rule #4: Raise Red Flags
❌ **Bad:** "I'll just implement it and hope it works."  
✅ **Good:** "This approach violates the encryption constraint in `techContext.md`. Can we discuss alternatives?"

### Rule #5: Document Trade-offs
❌ **Bad:** "We're using this library instead of the architecture plan."  
✅ **Good:** "We're using library X instead of Y because [justification]. This trade-off is documented in [file]."

---

## 📋 Memory Bank Update Protocol

**When to Update Memory Bank Files:**

### `projectbrief.md`
- When business requirements change
- When stakeholder information updates
- When success metrics need adjustment
- Frequency: Monthly review (Dr. Okonkwo)

### `techContext.md`
- When new architectural decisions are made
- When technology choices change
- When compliance requirements change
- Frequency: Every 2 weeks (Tech Lead)

### `progress.md`
- When sprint ends (Friday update)
- When blockers are resolved
- When timeline changes
- Frequency: Weekly updates (every Friday)

### `AGENTS.md`
- When new agent types are added
- When role responsibilities change
- When compliance constraints change
- Frequency: Quarterly review

---

## 🔗 Cross-Agent Communication

**When Agent A Needs Agent B:**

| Scenario | How to Escalate |
|---|---|
| Backend agent finds security issue | Tag Security Lead, reference `techContext.md` Section 6 |
| Frontend agent blocked by backend | Escalate to Tech Lead, include sprint deadline |
| ML agent needs data access permission | Request from Product Manager with use case |
| DevOps agent needs compliance approval | Request from Security Lead with justification |
| Any agent finds compliance gap | Escalate to Security Lead + Dr. Okonkwo |

---

## 📊 Daily Agent Standup Template

**Every agent** should provide this standup daily (or before shift end):

```markdown
## HealthCore Digital Agent Standup — [Agent Name]

**Date:** [YYYY-MM-DD]
**Sprint:** [Phase N, Sprint X]
**Current Task:** [What I'm working on]

### ✅ Completed Yesterday
- Task 1 (linked issue #123)
- Task 2 (PR #456)

### 🔄 In Progress Today
- Task 3 (60% complete)
- Task 4 (blocked by Task X)

### ⚠️ Blockers
- Blocker 1: [description] → Escalated to [person]
- Blocker 2: [description] → Mitigation: [plan]

### 📚 Memory Bank References Used
- `projectbrief.md` → [reason]
- `techContext.md` Section X → [reason]
- `progress.md` Sprint Y → [reason]

### 🎯 Plan for Tomorrow
- Task 5
- Task 6
```

---

## 🎓 Training & Onboarding for New Agents

**For every new agent joining HealthCore Digital:**

1. **Day 1: Business Context**
   - Read `projectbrief.md` (full)
   - Watch 30-min intro video (Dr. Okonkwo explaining clinic operations)
   - Chat with Product Manager about success metrics

2. **Day 2: Technical Foundation**
   - Read `techContext.md` (full)
   - Setup local development environment (Docker Compose)
   - Review live database schema

3. **Day 3: Current Status & Role**
   - Read `progress.md` (full)
   - Read AGENTS.md your role section (this file)
   - Pair program with agent already on team (4 hours)

4. **Day 4: First Task**
   - Assign "good first issue" (well-scoped, documented)
   - Mentor reviews work
   - Merge first PR (with guidance)

5. **Day 5: Team Sync**
   - Attend sprint planning
   - Present learnings to team
   - Ask questions in retro

---

## 🚨 When to Break These Rules

**Only when:**
1. **Explicit instruction from Dr. Okonkwo** (CEO) with written justification
2. **Patient safety at risk** (immediate action needed to prevent harm)
3. **Legal/compliance emergency** (threat of regulatory fine or breach)
4. **System outage** (production down, affecting patient care)

In any of these cases:
- Document the exception in a GitHub issue
- Label as `emergency` or `compliance-override`
- Brief the team at next standup
- Update relevant memory bank files

---

## 📞 Escalation Contacts

**Use Memory Bank First.** Only escalate if memory bank doesn't have answer:

| Escalation Type | Contact | Availability | Response Time |
|---|---|---|---|
| Technical/Architecture | James Osei (CTO) | Daily 10 AM - 6 PM UTC | <4 hours |
| Product/Prioritization | Product Manager (TBD) | Daily 9 AM - 5 PM UTC | <2 hours |
| Compliance/Security | Claire Whitfield | Daily 9 AM - 5 PM UTC (UK time) | <4 hours |
| Budget/Hiring/Major Decisions | Dr. Sandra Okonkwo | Weekly Friday 2 PM UTC | Weekly |
| Blockers / Emergencies | Slack #healthcore-dev | Real-time | Immediate |

---

## ✅ Agent Self-Check Checklist

Before starting a session, every agent should ask:

- [ ] Did I read the memory bank files for my role?
- [ ] Do I understand the current phase and sprint goals?
- [ ] Am I aware of compliance constraints specific to my work?
- [ ] Do I know who to escalate to if blocked?
- [ ] Is my work aligned with the roadmap in `progress.md`?
- [ ] Have I identified any red flags before starting?
- [ ] Do I know the success metrics for my current sprint?

**If any answer is "No," re-read the relevant section and schedule a sync with your mentor.**

---

## 🛑 Protected Files & Directories (Agents CANNOT Modify Without Explicit Confirmation)

**CRITICAL:** Agents must NOT modify these files/folders without explicit developer confirmation. Breaking this rule will block PRs and cause rework.

### 🔴 **Memory Bank Files** (NEVER modify without approval)

```
memory-bank/
├── projectbrief.md           🛑 LOCKED - Business truth
├── techContext.md            🛑 LOCKED - Architectural truth
├── progress.md               🛑 LOCKED - Sprint status truth
└── AGENTS.md                 🛑 LOCKED - Workflow rules
```

**Why:** These are the "source of truth" for the entire project.

**When to Modify:**
- `projectbrief.md` → Dr. Okonkwo (CEO) or Product Manager updates only
- `techContext.md` → James Osei (CTO) or Tech Lead updates only
- `progress.md` → Product Manager or Sprint Lead updates weekly
- `AGENTS.md` → James Osei (CTO) updates quarterly

**If Agent Needs Change:**
"I suggest updating `progress.md` Section X to reflect [reason]. This requires approval from [owner]."

---

### 🟠 **Workflow & Process Files** (Ask first, then modify)

```
docs/
├── COMMIT_WORKFLOW.md        🟠 Process file - confirm with James Osei
├── COMMIT_CHECKLIST.md       🟠 Process file - confirm with James Osei
└── WORKFLOW_SUMMARY.md       🟠 Process file - confirm with James Osei

CONTRIBUTING.md              🟠 Process file - confirm with James Osei
AGENTS.md                    🟠 (Also process file - locked, see above)
```

**Why:** These define HOW the team works together.

**When Agent Wants to Modify:**
"I found a process improvement: [idea]. Should I update CONTRIBUTING.md and COMMIT_WORKFLOW.md? Can I open a PR?"

**Approval Needed From:** James Osei (CTO)

---

### 🟡 **Configuration Files** (Read-only by default)

```
.github/
├── workflows/
│   ├── lint.yml              🟡 Read CI config, don't modify (James Osei approves)
│   ├── test.yml              🟡 Read CI config, don't modify (James Osei approves)
│   └── security.yml          🟡 Read CI config, don't modify (James Osei approves)

.git/
├── hooks/
│   └── pre-commit            🟡 Read automation, don't modify without approval

Configuration Files:
├── .flake8                   🟡 Python linting - ask before changing
├── .pylintrc                 🟡 Python linting - ask before changing
├── .prettierrc               🟡 TypeScript formatting - ask before changing
├── .eslintrc.json            🟡 TypeScript linting - ask before changing
├── pyproject.toml            🟡 Python config - ask before changing
├── tsconfig.json             🟡 TypeScript config - ask before changing
├── jest.config.js            🟡 Jest config - ask before changing
├── docker-compose.yml        🟡 Dev infrastructure - ask before changing
└── .gitignore               🟡 Git config - ask before changing
```

**Why:** Changing these affects THE ENTIRE TEAM'S development experience.

**When to Modify:**
- If linting rules are too strict/loose → Discuss in team retro
- If CI/CD is failing → Escalate to James Osei
- If Docker setup is broken → Report to DevOps lead
- If a new dependency needs adding → Add to requirements.txt, ask for config update

**Approval Needed From:**
- Linting config → Tech Lead (James Osei)
- CI/CD workflows → Tech Lead (James Osei)
- Docker/infra → DevOps engineer
- Dependencies → Tech Lead

---

### 🟢 **Code & Test Directories** (Modify freely as assigned)

```
services/api/
├── routers/                  ✅ Modify your assigned feature
├── models/                   ✅ Modify your assigned feature
├── schemas/                  ✅ Modify your assigned feature
├── tests/                    ✅ Modify your assigned feature
└── dependencies/             ✅ Modify your assigned feature

uis/backoffice/
├── src/components/           ✅ Modify your assigned feature
├── src/pages/                ✅ Modify your assigned feature
├── src/hooks/                ✅ Modify your assigned feature
└── src/__tests__/            ✅ Modify your assigned feature

agents/
├── tools/                    ✅ Modify your assigned feature
├── skills/                   ✅ Modify your assigned feature
└── tests/                    ✅ Modify your assigned feature

packages/shared/              ✅ Modify shared types (coordinate with team)

data/                         ✅ Add datasets as needed
```

**Rule:** You can modify code in directories assigned to your sprint.

**But Check First:**
- If you modify `packages/shared/` types → Other components may break
  - Solution: Coordinate with other teams, provide migration plan
- If you modify core utilities → Check tests don't break across components
  - Solution: Run full test suite before PR

---

### 🚫 **ABSOLUTELY FORBIDDEN** (Never touch without escalation)

```
1. Git history (never force push)
   ❌ git push --force
   ❌ git reset --hard
   ✅ Always create new commits

2. Production secrets (never commit)
   ❌ Database passwords in code
   ❌ API keys in .env files (checked in)
   ❌ JWT secrets in repositories
   ✅ Use environment variables

3. Database migrations in production
   ❌ Modify migration files after deployed
   ✅ Create new migration file for changes

4. Direct patient data access
   ❌ Query patient data without audit logging
   ❌ Export PHI to local machine
   ✅ Use secure audit-logged APIs

5. Modifying other's work without permission
   ❌ Rewriting another developer's code without PR
   ✅ Suggest changes in code review
```

---

## 📋 Agent Checklist Before Modifying Protected Files

**Before touching ANY protected file, ask:**

```
1. Is this file in the 🛑 LOCKED category?
   ☐ YES → STOP, escalate to file owner
   ☐ NO → Continue to step 2

2. Is this file in the 🟠 PROCESS category?
   ☐ YES → Ask James Osei for approval before modifying
   ☐ NO → Continue to step 3

3. Is this file in the 🟡 CONFIG category?
   ☐ YES → Ask relevant lead (Tech Lead, DevOps, etc.)
   ☐ NO → Continue to step 4

4. Is this file in the 🟢 CODE category?
   ☐ YES → Modify freely if assigned to this feature
   ☐ NO → Something's wrong, escalate

5. Is this in the 🚫 FORBIDDEN category?
   ☐ YES → DO NOT MODIFY, escalate immediately
   ☐ NO → Safe to proceed

Before submitting PR:
☐ No protected files modified without approval
☐ Commit message explains changes
☐ All tests pass
☐ No secrets exposed
```

---

## 🔄 How Agents Should Propose Changes to Protected Files

**Scenario:** Agent notices linting rules could be improved.

**WRONG WAY:**
```
Agent commits to .eslintrc.json without asking
  ↓
PR submitted
  ↓
Code review: "Why did you change linting config?"
  ↓
PR rejected
  ↓
Agent rewrites same PR
  ↓
Time wasted
```

**RIGHT WAY:**
```
Agent: "I notice .eslintrc.json could enforce better practices 
       for React hooks. Can I propose a change? 
       Should I open an issue first or PR?"

Tech Lead: "Great idea! Open a GitHub issue with the specific 
           changes you'd recommend. We can discuss in next retro."

Issue Discussion: "Let's make these 3 changes. Test locally first?"

Agent submits PR with approval from Tech Lead
  ↓
PR merged smoothly
```

---

## 📞 Who Owns Protected Files?

| File/Directory | Owner | Approval Required | Contact |
|---|---|---|---|
| `memory-bank/projectbrief.md` | Product Manager / Dr. Okonkwo | Yes | Slack #healthcore-dev |
| `memory-bank/techContext.md` | James Osei (CTO) | Yes | James Osei |
| `memory-bank/progress.md` | Product Manager | Yes | Product Manager |
| `memory-bank/AGENTS.md` | James Osei (CTO) | Yes | James Osei |
| Workflow docs (CONTRIBUTING.md, etc.) | James Osei (CTO) | Yes | James Osei |
| CI/CD workflows (.github/workflows/) | James Osei (CTO) | Yes | James Osei |
| Linting config (.eslintrc.json, etc.) | James Osei (CTO) | Yes | James Osei |
| Docker config (docker-compose.yml) | DevOps Engineer | Yes | Assigned DevOps lead |
| Feature code (services/, uis/, etc.) | Sprint Lead / Tech Lead | No (assigned work) | Your team |

---

**Last Reviewed:** November 17, 2026  
**Next Review:** February 17, 2027 (Quarterly)  
**Owner:** James Osei (CTO)

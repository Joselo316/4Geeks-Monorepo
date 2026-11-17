# Contributing to HealthCore Digital

**Version:** 1.0  
**Last Updated:** November 17, 2026

Welcome to the HealthCore Digital team! This guide explains how to contribute code, report issues, and work collaboratively.

---

## 🚀 Before You Start

**Read these files in order:**

1. [`memory-bank/projectbrief.md`](/workspaces/4Geeks-Monorepo/memory-bank/projectbrief.md) — Business context
2. [`memory-bank/techContext.md`](/workspaces/4Geeks-Monorepo/memory-bank/techContext.md) — Architecture & tech stack
3. [`memory-bank/AGENTS.md`](/workspaces/4Geeks-Monorepo/memory-bank/AGENTS.md) — Your role & responsibilities
4. [`docs/COMMIT_WORKFLOW.md`](/workspaces/4Geeks-Monorepo/docs/COMMIT_WORKFLOW.md) — Workflow before every commit

---

## 📋 Development Setup

### Prerequisites

- Python 3.11+
- Node.js 18+
- Docker & Docker Compose
- Git

### Local Environment

```bash
# Clone repo
git clone https://github.com/Joselo316/4Geeks-Monorepo.git
cd 4Geeks-Monorepo

# Create Python virtual environment
python3.11 -m venv venv
source venv/bin/activate  # Linux/Mac
# or: venv\Scripts\activate  # Windows

# Install backend dependencies
pip install -r services/api/requirements.txt

# Install frontend dependencies
cd uis/backoffice
npm install
cd ../..

# Start Docker Compose
docker-compose up
```

### Verify Setup

```bash
# Backend health check
curl http://localhost:8000/docs  # FastAPI Swagger UI

# Frontend health check
curl http://localhost:3000       # React dev server

# Database health check
psql -U healthcore -d healthcore_db -c "SELECT 1"
```

---

## 🔄 Workflow: From Issue to Merged PR

### 1. Find or Create an Issue

**All work must have a GitHub issue.** 

- [ ] Issue exists in backlog
- [ ] Issue has clear description (what, why, acceptance criteria)
- [ ] Issue has labels (priority, phase, component)
- [ ] Issue is assigned to you
- [ ] Issue is in current sprint (if applicable)

**Create issue if needed:**
```markdown
Title: [Brief description]

Description:
- What needs to be done
- Why it matters
- Who it affects

Acceptance Criteria:
- [ ] Criterion 1
- [ ] Criterion 2

Related: [Links to other issues/PRs]
```

### 2. Create Feature Branch

```bash
# Update develop branch
git checkout develop
git pull origin develop

# Create feature branch from develop
git checkout -b feature/your-feature-name

# Examples:
git checkout -b feature/patient-search
git checkout -b fix/appointment-timezone
git checkout -b refactor/claim-schema
```

**Branch Naming Convention:**
- `feature/[description]` — New feature
- `fix/[description]` — Bug fix
- `refactor/[description]` — Code restructuring
- `docs/[description]` — Documentation
- `perf/[description]` — Performance improvement

### 3. Write Code (and Commit Frequently)

Make small, logical commits as you work:

```bash
# Edit files
# ... your changes ...

# Check status
git status

# Stage changes
git add services/api/routers/patients.py

# Commit with message
git commit -m "feat(patients): add patient search endpoint

- Implement /patients/search with fuzzy matching
- Add clinic_id filtering for data residency
- Include audit logging for HIPAA compliance
- Tests: 95% coverage with 12 test cases"
```

**Commit Often:** Better to have 10 small commits than 1 massive commit. We'll squash before merging.

### 4. Before Each Commit: Run the 4-Step Workflow

**THIS IS MANDATORY.** See [`docs/COMMIT_WORKFLOW.md`](/workspaces/4Geeks-Monorepo/docs/COMMIT_WORKFLOW.md):

```bash
# STEP 1: Code Quality
black services/api --line-length 100
ruff check services/api --fix
prettier uis/backoffice/src --write
eslint uis/backoffice/src --fix
mypy services/api
tsc --noEmit uis/backoffice

# STEP 2: Tests
pytest services/api/tests -v --cov=services/api --cov-fail-under=80
npm run test:unit -- --coverage uis/backoffice

# STEP 3: Security
bandit -r services/api --severity-level medium
safety check services/api/requirements.txt
npm audit uis/backoffice
git secrets --scan

# STEP 4: (Before PR) Code review, documentation, commit message
```

**All must pass before you push.**

### 5. Push to Remote

```bash
# Push to your feature branch
git push origin feature/your-feature-name

# If branch didn't exist, Git will create it
# After first push, use: git push
```

### 6. Create Pull Request

Go to GitHub and create PR:

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

## Notes for Reviewer
[Anything special to review]
```

### 7. Code Review & Iteration

- Wait for 2 approvals (1 peer + 1 tech lead)
- Address feedback in new commits (don't amend history)
- Re-request review after changes
- Once approved, Tech Lead merges

### 8. Merge & Deploy

**Tech Lead handles merge:**
```bash
# Rebase on develop (no merge commits)
git rebase develop

# Squash into 1 commit
git rebase -i develop

# Push to develop
git push origin develop

# Auto-delete feature branch
```

**CI/CD then deploys to staging automatically.**

---

## 🧪 Testing Guide

### Unit Tests (Python)

```bash
# Run all backend tests
pytest services/api/tests -v

# Run specific test file
pytest services/api/tests/test_patients.py -v

# Run with coverage
pytest services/api/tests --cov=services/api --cov-report=html

# Run in watch mode
pytest-watch services/api/tests
```

**Write tests alongside code:**
```python
# services/api/tests/test_patients.py

import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_patient_search():
    response = client.get("/patients/search?name=John&clinic_id=1")
    assert response.status_code == 200
    assert len(response.json()) > 0

def test_patient_search_unauthorized():
    response = client.get("/patients/search?name=John")
    assert response.status_code == 401  # Requires auth
```

### Unit Tests (TypeScript)

```bash
# Run all frontend tests
npm run test:unit

# Run specific test file
npm run test:unit -- PatientSearch.test.tsx

# Run with coverage
npm run test:unit -- --coverage

# Run in watch mode
npm run test:unit -- --watch
```

**Write tests:**
```typescript
// uis/backoffice/src/components/__tests__/PatientSearch.test.tsx

import { render, screen, fireEvent } from '@testing-library/react';
import PatientSearch from '../PatientSearch';

describe('PatientSearch', () => {
  it('displays search results', () => {
    render(<PatientSearch />);
    
    const input = screen.getByPlaceholderText('Search patients...');
    fireEvent.change(input, { target: { value: 'John' } });
    
    expect(screen.getByText(/John Doe/)).toBeInTheDocument();
  });
});
```

### Integration Tests

```bash
# Tests that touch DB, APIs, etc
pytest services/api/tests/integration -v
```

### E2E Tests (Cypress)

```bash
# Run E2E tests headless
npm run test:e2e

# Run E2E tests with UI (interactive)
npm run test:e2e -- --no-headless
```

**Write E2E tests for critical flows:**
```javascript
// uis/backoffice/cypress/e2e/appointment-booking.cy.js

describe('Appointment Booking', () => {
  it('patient can book appointment', () => {
    cy.login('clinic-admin@healthcore.com', 'password');
    cy.visit('/appointments/new');
    
    cy.get('[data-testid="patient-search"]').type('John');
    cy.get('[data-testid="patient-option"]').first().click();
    
    cy.get('[data-testid="date-picker"]').type('2024-12-20');
    cy.get('[data-testid="time-picker"]').select('10:00 AM');
    
    cy.get('[data-testid="submit-button"]').click();
    
    cy.contains('Appointment booked successfully').should('be.visible');
  });
});
```

---

## 🐛 Reporting Issues

Found a bug? Create an issue:

```markdown
Title: [Bug] Brief description

Reproduction Steps:
1. Go to...
2. Click on...
3. Expected: ...
4. Actual: ...

Environment:
- Python version: 3.11
- Node version: 18.x
- Browser: Chrome 120
- OS: macOS 14

Screenshots/Logs:
[Attach relevant output]
```

---

## 📚 Code Style Guide

### Python

- Use type hints for all function arguments and returns
- Max line length: 100 characters
- Use `snake_case` for variables and functions
- Use `PascalCase` for classes
- Docstrings: Google style (required for public functions)

```python
def search_patients(name: str, clinic_id: int) -> List[Patient]:
    """Search patients by name within a specific clinic.
    
    Args:
        name: Patient name to search for (fuzzy matching)
        clinic_id: Clinic to scope search to
        
    Returns:
        List of matching Patient objects
        
    Raises:
        PermissionError: If user not authorized for clinic_id
    """
    # Implementation
```

### TypeScript

- Use type annotations (not `any`)
- Max line length: 100 characters
- Use `camelCase` for variables and functions
- Use `PascalCase` for classes and components
- Prefer interfaces over types (for class definitions)

```typescript
interface PatientSearchParams {
  name: string;
  clinicId: number;
}

export function searchPatients(
  params: PatientSearchParams
): Promise<Patient[]> {
  // Implementation
}
```

### Documentation

- Every API endpoint needs docstring (FastAPI auto-generates Swagger)
- Every component needs JSDoc comment (React)
- Complex algorithms need inline comments
- Keep README.md updated

---

## 🤝 Code Review Standards

**As a Reviewer:**
- [ ] Code solves the stated problem
- [ ] Tests are adequate (>80% backend, >70% frontend)
- [ ] No security vulnerabilities
- [ ] Follows style guide
- [ ] No hardcoded credentials or PHI
- [ ] Audit logging for data access
- [ ] Performance is acceptable
- [ ] Documentation updated

**Comment Helpfully:**
```
// Good
This function doesn't have a return type annotation. 
According to our style guide (see CONTRIBUTING.md), 
all functions need type hints.

Suggestion:
def search_patients(name: str, clinic_id: int) -> List[Patient]:
    ...

// Bad
Add types bro
```

**Approve When Confident:**
Only click "Approve" if you're confident in the code quality.

---

## 🚀 Deployment

### Staging

```bash
# Automatic when PR is merged to develop
# Check status: https://github.com/Joselo316/4Geeks-Monorepo/actions
```

### Production

```bash
# Manual trigger when release is ready
# Tech Lead tags release and merges to main
# Deployment via: https://github.com/Joselo316/4Geeks-Monorepo/actions
```

---

## ❓ Questions?

- **Technical:** Slack #healthcore-dev or @James Osei
- **Process:** This file (CONTRIBUTING.md)
- **Business:** memory-bank/ files
- **Escalation:** Dr. Sandra Okonkwo (CTO: James Osei)

---

## 📋 Contributor Agreement

By contributing to HealthCore Digital, you agree to:

✅ Follow the memory bank guidance (`projectbrief.md`, `techContext.md`, `AGENTS.md`)  
✅ Complete the 4-step commit workflow (`docs/COMMIT_WORKFLOW.md`)  
✅ Write tests for new code (>80% backend, >70% frontend)  
✅ Respect HIPAA and UK GDPR compliance requirements  
✅ Participate in code reviews professionally  
✅ Keep documentation updated  
✅ Report issues early, don't hide them  

**Thank you for building HealthCore Digital! 🎉**

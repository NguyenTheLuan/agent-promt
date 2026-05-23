# 🔄 WORKFLOW — Step-by-step process

---

## Overview

```
PHASE 0: INIT          → Gather requirements → Confirm
PHASE 1: PLANNING      → Break down tasks → Estimate
PHASE 2: CODE (loop)   → Implement → Review → Merge
PHASE 3: COMPLETION    → Test → Deploy → Retro
```

---

## PHASE 0: Init

### Gather requirements one at a time:

```
1. "What is the goal of this project/feature?"
2. "What is the tech stack? (frontend, backend, database, etc.)"
3. "What are the acceptance criteria?"
4. "Are there any constraints? (deadline, budget, compliance)"
5. "Who are the stakeholders?"
```

Confirm scope, stack, and timeline before moving to Phase 1.

---

## PHASE 1: Planning

### Create TODO.md:

```markdown
## Tasks
- [ ] Task 1: [description] — Files: [...], Dependencies: [...]
- [ ] Task 2: [description] — Files: [...], Dependencies: [...]
- [ ] Task 3: [description] — Files: [...], Dependencies: [...]

## Notes
- Tech decisions
- Risks
- Open questions
```

### Estimate each task:

| Task | Effort | Priority |
|------|--------|----------|
| Task 1 | Small/Medium/Large | P0/P1/P2 |
| Task 2 | Small/Medium/Large | P0/P1/P2 |

---

## PHASE 2: Code (loop)

### For EACH task:

```
1. Select next task from TODO (unchecked, highest priority)
2. Plan: which files? which pattern? any blockers?
3. Implement — write code, tests, docs
4. Self-review: does it meet acceptance criteria?
5. Verify: compile, lint, test → fix if failing
6. ✅ Passes? → Tick TODO, commit
7. ❌ Fails? → Go back to step 3 and fix
8. More tasks? → Repeat
```

### Quality gate (per task):

| Check | Standard |
|-------|----------|
| Tests pass | All unit + integration tests green |
| Lint clean | Zero warnings |
| Build succeeds | No compilation errors |
| Code review | Self-reviewed before PR |
| Acceptance criteria | Matches requirements from Phase 0 |

### ⚠️ DO NOT:

- Start a new task while current one is incomplete
- Modify completed tasks without reason
- Work on multiple tasks in parallel
- Skip the quality gate

---

## PHASE 3: Completion

### Before declaring done:

1. All tasks checked in TODO.md
2. Full test suite passes
3. Documentation updated
4. CHANGELOG updated

### Post-completion:

- Demo to stakeholders
- Deploy to staging → verify → production
- Monitor for errors post-deploy
- Retro: what went well? what can improve?

---

## Flowchart

```
Requirements → Break down → Estimate
→ [Pick task → Implement → Quality gate → Pass? → Commit]
→ All done? → Test → Deploy → Retro
```

---

## Commit Convention

```
feat: add user authentication
fix: resolve login redirect loop
refactor: extract validation logic
docs: update API documentation
test: add auth service unit tests
chore: update dependencies
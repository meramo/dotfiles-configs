# Workflow Orchestration

## 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

## 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

## 3. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

## 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness
- **Before every git commit**, run the project's CI checks locally (lint, typecheck, tests). Never let broken code reach GitHub Actions. If the project has a `precommit` script, run it. If not, run `npm run lint && npx tsc --noEmit && npm test -- --bail` (or equivalent).

## 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

## 6. Autonomous Bug Fixing

- When given a bug report, just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

# Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

# React / React Native Guardrails

## State Initialization
- **Never leave parent state as `null` when data is available to initialize it.** If a parent component has data (e.g., `cards` from a hook), initialize derived state eagerly via `useEffect` in the parent — don't rely on a child component's effect to call back and set it. Child effects fire after render, creating a visible gap where dependent UI is missing.
- **Don't use derived booleans as useEffect dependencies.** `[items.length > 0]` collapses all non-empty arrays into `true`, so the effect won't re-fire when the array changes identity. Use the actual data reference: `[items]`.

## Verify the Mount Path
- **Always mentally trace the first render frame.** The initial mount sequence (loading → content → effects fire → re-render) is the most fragile and the first thing users see. Ask: "What does the user see on frame 1 after content appears?" If any UI depends on state set by effects, there will be a flash of missing content.
- **Test initial load, not just steady-state.** Bugs that only manifest during the first few frames after mount are easy to miss when reasoning about the "user interacts → state updates → UI updates" steady-state flow.

# Pre-Commit Hook Standard

**For every project**, set up a git pre-commit hook that mirrors CI. This is non-negotiable — broken code should never reach GitHub Actions.

## Setup (do this when starting work on any new project)

1. Check if `.github/workflows/` exists — read the CI config to know what checks run
2. Create `.git/hooks/pre-commit` (executable) that runs the same checks
3. Add a `precommit` script to `package.json` for manual runs

## Template

```bash
#!/bin/bash
# Pre-commit hook: mirrors CI checks locally
set -e

echo "🔍 Pre-commit checks..."

# 1. Lint (adjust command per project)
echo "━━━ Lint ━━━"
npm run lint 2>&1
echo "✅ Lint passed"

# 2. Type check (TypeScript projects)
echo "━━━ TypeScript ━━━"
npx tsc --noEmit 2>&1
echo "✅ TypeScript passed"

# 3. Tests
echo "━━━ Tests ━━━"
npm test -- --bail --silent 2>&1
echo "✅ Tests passed"

echo "✅ All pre-commit checks passed!"
```

Adjust commands to match whatever CI runs (e.g., `pytest` for Python, `go vet && go test` for Go). The principle is: **if CI checks it, the hook checks it**.

# Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

# Git Identity

- **Always commit as:** Igor Kandyba <igor.kandyba@gmail.com>
- **Never** add `Co-Authored-By` trailers or use any other author identity
- If in doubt about which identity to use, **ask the user before committing**

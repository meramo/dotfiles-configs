Behavioral guidelines for all projects. These bias toward caution over speed — for trivial tasks, use judgment. Project-specific instructions should be layered on top via project `CLAUDE.md` files.

# Workflow Orchestration

## 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

## 2. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 3. Subagent Strategy

- Use subagents liberally to keep the main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

## 4. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for the relevant project

## 5. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 6. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness
- **Before every git commit**, run the project's CI checks locally (lint, typecheck, tests). Never let broken code reach CI. If the project has a `precommit` script, run it; otherwise run the equivalent checks for the stack.
- **Verify write-read symmetry**: After implementing any write path (API POST, file write, DB insert), immediately trace the full read path that should surface the data. Confirm no filters drop it, no caches hide it, no format mismatches lose it. A successful write means nothing if the data never appears.

## 7. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

## 8. Autonomous Bug Fixing

- When given a bug report, just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

# Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

# Pre-Commit Hooks

For every project, set up a git pre-commit hook that mirrors CI. Broken code should never reach CI.

1. Inspect the project's CI config to learn what checks run
2. Create an executable `.git/hooks/pre-commit` that runs the same checks
3. Expose a `precommit` script (or equivalent) for manual runs

The principle: **if CI checks it, the hook checks it.** Adapt commands to the stack (npm, pytest, go test, cargo, etc.).

# Data Integrity Guardrails

## Write-Read Symmetry
Every data write MUST have a verified read path. Before declaring a feature done:
1. **Trace the round-trip**: Data written → storage → query/filter → API response → UI render. Walk every step.
2. **Check for silent filters**: Scan for any filtering, conditional skips, required-field checks, cache TTLs, or format checks between the write and the read. Any of these can silently drop data.
3. **Verify across storage layers**: If the system has multiple stores (cache + DB, local + remote, primary + replica), verify the data appears in all of them. A local write that skips the remote store means the deployed system sees nothing.
4. **Verify the UI, not just the API**: A successful write response is not proof the feature works. Confirm the data renders where the user expects it — including the right bucket, group, or filter.

## New Format Compatibility
When introducing a new data format or relaxing constraints:
- **Audit all consumers**: Find every place the old format is read, filtered, or parsed. Each one is a potential silent failure.
- **Check serialization boundaries**: If state is persisted (storage, cookies, caches, config), ensure old serialized values don't break with the new format. Add validation on deserialization.
- **Existing query params and flags**: A new write format that bypasses an existing read-side filter is the #1 cause of "data disappears" bugs.

# Core Principles

## Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked
- No abstractions for single-use code
- No "flexibility" or "configurability" that wasn't requested
- No error handling for impossible scenarios
- If you write 200 lines and it could be 50, rewrite it

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting
- Don't refactor things that aren't broken
- Match existing style, even if you'd do it differently
- If you notice unrelated dead code, mention it — don't delete it

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused
- Don't remove pre-existing dead code unless asked

The test: every changed line should trace directly to the user's request.

## No Laziness

Find root causes. No temporary fixes. Senior developer standards.

# Git Identity

- **Always commit as:** Igor Kandyba <igor.kandyba@gmail.com>
- **Never** add `Co-Authored-By` trailers or use any other author identity
- If in doubt about which identity to use, **ask the user before committing**

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---
name: implement-feature
description: TDD workflow for implementing a new feature — understand, plan, types, red, green, verify, checklist.
---

# Implement Feature Workflow

## 1. Understand

- Read the feature spec or user description
- Read CLAUDE.md and `.claude/rules/` for relevant constraints
- If the project has domain documentation, read the relevant parts
- Ask clarifying questions before starting if the scope is ambiguous

## 2. Plan

Switch to Plan Mode before writing any code.

- Identify affected packages, modules, and feature areas
- List files to create/modify (be specific)
- Estimate line counts — if any file will exceed 200 lines, plan the split NOW
- Get user approval before proceeding

This step exists because agents tend to start coding before fully understanding the problem. Planning forces understanding.

## 3. Types First

Define or update TypeScript types before writing implementation.

- New domain types go in the most appropriate package (core types, feature-local types, etc.)
- Commit types before implementation so other modules can depend on them
- Running typecheck after defining types confirms the shape is coherent

## 4. Write Failing Tests (Red)

Write tests that define the expected behavior **before writing implementation**.

- Tests should describe behavior, not implementation
- Write both happy path AND error cases
- Run the tests — confirm they FAIL
- If tests pass before implementation exists, they are testing nothing — rewrite them

This step is non-negotiable. Without failing tests first, you don't know what you're building.

## 5. Implement (Green)

Write the minimum code to make the tests pass.

- No extra features. No premature abstractions. (YAGNI)
- If you're writing code that no test exercises, stop and ask why

## 6. Verify

Run all checks:

```bash
# Adapt to your project's scripts
npx tsc --noEmit    # typecheck
npx vitest run      # all tests
```

Fix any failures before proceeding.

## 7. Review Checklist

- [ ] All new files under 200 lines? (300 hard max)
- [ ] Tests cover happy path AND error cases?
- [ ] No `any` types added?
- [ ] No hardcoded values that should be constants or config?
- [ ] Imports respect package boundaries?
- [ ] No unused imports or dead code?
- [ ] No `it.skip` or `test.skip`?

## 8. Commit

Use conventional commit message: `feat: [description of feature]`

If the feature is large, make multiple logical commits (types, tests, implementation, etc.) rather than one giant commit.

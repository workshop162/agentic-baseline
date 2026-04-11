---
name: fix-issue
description: Debug and fix a bug — reproduce with a failing test, find root cause, minimal fix, verify.
---

# Fix Issue Workflow

## 1. Reproduce

Write a failing test that demonstrates the bug. If you can't write a test, you don't understand the bug yet.

Run the test. Confirm it FAILS with the expected error. A test that passes before you fix anything is not a test — it's noise.

## 2. Investigate

- Read the failing code path end-to-end
- Use search tools to find related usages if the root cause isn't obvious
- Check recent git changes: `git log --oneline -10 -- path/to/file`

Do NOT guess. Trace the actual execution path.

## 3. Root Cause

Identify the exact line(s) causing the issue. Write down the root cause before touching any code.

Common root causes across TypeScript projects:
- Off-by-one in array/index handling
- Async race conditions (missing await, incorrect Promise chaining)
- Stale cached data after a mutation
- Type coercion bugs hidden by `as any`
- Missing null/undefined guard on optional data

## 4. Fix

Make the minimal change to fix the root cause.

- Do NOT refactor surrounding code
- Do NOT add "while I'm here" improvements
- Do NOT add extra error handling beyond what the bug requires

The fix should be the smallest possible change that makes the failing test pass.

## 5. Verify

```bash
# Adapt to your project's scripts
npx tsc --noEmit    # typecheck
npx vitest run      # tests
```

Confirm:

- The failing test now passes
- No other tests broke (no regressions)
- Typecheck is clean

## 6. Commit

Use conventional commit: `fix: [description of what was fixed]`

Reference the issue number if one exists: `fix: handle null user in auth middleware (#42)`

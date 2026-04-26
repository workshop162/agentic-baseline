# /deep-audit Command

Perform a comprehensive code audit of the specified area of the codebase.

## Usage

`/deep-audit [area]` — e.g., `/deep-audit src/`, `/deep-audit packages/api`, `/deep-audit apps/web`

If no area is specified, audit the entire project starting with the most critical paths.

## Audit Process

1. **Read AGENTS.md first** to understand architecture, conventions, and package boundaries
2. **Systematically review** the target area, focusing on:
   - Security vulnerabilities (auth bypasses, injection, exposed secrets, missing authorization)
   - Performance issues (N+1 queries, missing indexes, unnecessary re-renders, bundle size)
   - Error handling gaps (unhandled promises, missing try/catch, silent failures)
   - Architecture violations (wrong package boundaries per AGENTS.md rules)
   - Dead code and unused imports
   - Type safety gaps (`any` types, missing null checks, unsafe casts)
   - Missing or broken validation (schemas not matching actual usage)
   - Inconsistent patterns (doing the same thing different ways across the codebase)
   - Accessibility issues in UI code (missing labels, keyboard nav, screen reader support)
   - Test coverage gaps in critical paths

3. **Write ALL findings** to a dated audit file:
   - Path: `[project-local-audit-dir]/audit-[area-kebab]-[YYYY-MM-DD].md`
   - If no audit directory is configured, ask the user where to put it

## Output Format

Write findings to the audit file using this structure:

```markdown
# Code Audit: [Area]

**Date**: YYYY-MM-DD
**Scope**: [what was reviewed]
**Summary**: [1-2 sentence overview]

## Critical (fix immediately)

### [C1] Title

- **File**: `path/to/file.ts:lineNumber`
- **Issue**: What's wrong
- **Risk**: What could go wrong if not fixed
- **Fix**: Specific description of the fix needed
- **Verification**: How to confirm it's fixed (test command, manual check, etc.)
- **Status**: pending

## High (fix soon)

### [H1] Title

- **File**: `path/to/file.ts:lineNumber`
- **Issue**: ...
- **Risk**: ...
- **Fix**: ...
- **Verification**: ...
- **Status**: pending

## Medium (improve when touching this code)

### [M1] Title

...

## Low (nice to have)

### [L1] Title

...

## Architecture Notes

Any broader observations about patterns, tech debt trends, or structural concerns that aren't tied to a single file.
```

## Rules

- **Be specific** — include file paths and line numbers, not vague observations
- **Be actionable** — every finding must have a concrete fix description
- **Be honest about severity** — don't inflate everything to critical
- **Include verification steps** — how to confirm the fix worked (run a test, check a behavior, typecheck, etc.)
- **Don't list style nits** — focus on things that affect correctness, security, performance, or maintainability
- **Group related issues** — if 10 files have the same problem, that's ONE finding with a list of affected files
- **Cap at ~30 findings** — if there are more, prioritize the top 30 and note that more exist

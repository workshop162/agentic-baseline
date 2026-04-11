# /refactor Command

Perform a code quality and optimization review of the specified area. Unlike `/deep-audit` (which focuses on security and correctness), this command focuses on making the code cleaner, faster, and more maintainable.

## Usage

`/refactor [area]` — e.g., `/refactor src/`, `/refactor packages/api`, `/refactor apps/web/components`

You can target broad areas or drill into specific directories.

## Review Focus Areas

1. **Read CLAUDE.md first** to understand architecture and conventions

2. **Systematically review** the target area for:

### Efficiency

- Unnecessary re-renders (missing memo, useMemo, useCallback where it matters)
- Redundant API calls (same data fetched in multiple places)
- N+1 query patterns in data access layers
- Large bundle imports that could be lazy-loaded or tree-shaken
- Expensive computations that should be cached
- Database queries missing indexes or doing full table scans

### Simplification

- Overly complex functions that should be broken up (>50 lines)
- Deep nesting (>3 levels of if/else or callbacks)
- Copy-pasted logic that should be extracted into shared utilities
- Abstractions that add complexity without value (over-engineering)
- Components or modules doing too many things (violating single responsibility)

### Modernization

- Old patterns that have better modern alternatives
- Manual implementations of things libraries already handle
- Callback-style async that should be async/await
- Deprecated API usage

### DRY Violations

- Duplicated business logic across files
- Similar components that could share a base
- Repeated validation logic that should use shared schemas
- Copy-pasted error handling patterns

### Code Organization

- Files in the wrong directory per CLAUDE.md rules
- Circular dependencies
- Files over 300 lines that should be split
- Unclear naming (functions, variables, files)
- Missing or misleading TypeScript types

3. **Write ALL findings** to a dated refactor file:
   - Path: `[project-local-audit-dir]/refactor-[area-kebab]-[YYYY-MM-DD].md`
   - If no audit directory is configured, ask the user where to put it

## Output Format

```markdown
# Refactor Review: [Area]

**Date**: YYYY-MM-DD
**Scope**: [what was reviewed]
**Summary**: [1-2 sentence overview]
**Estimated Total Effort**: [X hours]

## Quick Wins (< 15 min each)

### [Q1] Title

- **File**: `path/to/file.ts:lineNumber`
- **What**: What can be improved
- **Why**: What benefit this gives (performance, readability, maintainability)
- **How**: Specific refactor steps
- **Status**: pending

## Moderate Refactors (15 min - 1 hour each)

### [R1] Title

- **File**: `path/to/file.ts` (or multiple files)
- **What**: What can be improved
- **Why**: What benefit this gives
- **How**: Specific refactor steps
- **Risk**: What could break (low/medium/high)
- **Verification**: How to confirm nothing broke
- **Status**: pending

## Larger Restructuring (1+ hours each)

### [L1] Title

- **Files**: List of affected files
- **What**: What can be improved
- **Why**: What benefit this gives
- **How**: Step-by-step approach
- **Risk**: What could break
- **Dependencies**: What else needs to change
- **Verification**: How to confirm nothing broke
- **Status**: pending

## Patterns & Observations

Broader notes on code patterns, consistency, tech debt trends, or architectural suggestions that don't map to a single finding.
```

## Rules

- **Be specific** — include file paths, line numbers, and concrete before/after examples where helpful
- **Prioritize impact** — a refactor that simplifies 10 files matters more than one that saves 2 lines
- **Don't be pedantic** — skip style preferences, bikeshedding, and formatting opinions
- **Consider risk** — a refactor that could break production is different from one that's purely additive
- **Group related items** — if the same pattern appears in 8 files, that's ONE finding
- **Show the win** — explain what gets better (faster, fewer lines, easier to test, etc.)
- **Cap at ~25 findings** — prioritize the highest-impact items
- **Don't duplicate the audit** — skip security issues; those belong in `/deep-audit`

---
name: review
description: Quality review checklist for any set of changes — typecheck, tests, line counts, types, naming, boundaries.
---

# Review Checklist

Pass a feature, directory, or list of files to review.

This review catches the most common agent mistakes: type errors, untested code, oversized files, `any` types, naming violations, and boundary violations.

## 1. Typecheck

```bash
npx tsc --noEmit
```

All errors must be resolved. Do not proceed until clean.

## 2. Tests

```bash
npx vitest run
```

All tests must pass. No skipped tests (`it.skip` and `test.skip` are forbidden).

If tests were added: confirm the tests actually test behavior, not just mirror implementation.

## 3. Line Count Audit

Check every `.ts`/`.tsx` file touched in the changes.

- Flag files over 200 lines (WARNING — consider splitting)
- Hard stop on any non-test file over 300 lines (VIOLATION — must split)
- Test files: 500-line hard max

Run to find violations:
```bash
find src -name "*.ts" -o -name "*.tsx" | grep -v node_modules | grep -v dist | grep -v ".test." | xargs wc -l | sort -rn | head -20
```

## 4. `any` Type Audit

Search for `: any` and `as any` in the changed files.

```bash
grep -rn ": any\|as any" src --include="*.ts" --include="*.tsx" | grep -v ".test."
```

Every `any` is a violation unless it has an explicit documented exception (e.g., a third-party library boundary).

## 5. Naming Convention Check

Adapt to your project's conventions. Common TypeScript conventions:
- Components: `PascalCase.tsx`
- Hooks: `use-kebab-case.ts` or `useCamelCase.ts`
- Utils/services: `kebab-case.ts`
- Tests: `*.test.ts` or `*.test.tsx`

Check that new files follow the existing convention in their directory.

## 6. Import Boundary Check

Verify all imports respect the dependency graph defined in AGENTS.md.

Look for:
- Imports that go "up" in the dependency tree (lower-level packages importing from higher-level)
- Cross-domain imports that should go through a shared interface
- Direct imports of internal modules that should use public package APIs

## 7. Dead Code Check

Scan for:
- Unused imports
- Functions defined but never called
- Variables declared but never used
- Commented-out code blocks

## Output Format

```
ITEM 1 (Typecheck): PASS / FAIL — [error count if failed]
ITEM 2 (Tests): PASS / FAIL — [X passed, Y failed]
ITEM 3 (Line Count): PASS / WARNING — [list files over 200] / FAIL — [list files over 300]
ITEM 4 (Any Types): PASS / FAIL — [X violations, list them]
ITEM 5 (Naming): PASS / FAIL — [list violations]
ITEM 6 (Imports): PASS / FAIL — [list violations]
ITEM 7 (Dead Code): PASS / WARNING — [list findings]
```

All items must be PASS before merging or committing. Address FAIL items immediately. WARNING items should be tracked if not fixed.

# Coding Standards for Agentic Development

This codebase is developed entirely by AI agents. These standards optimize for agent edit accuracy and code maintainability.

## File Size Limits

| Type             | Soft Limit    | Hard Max  |
| ---------------- | ------------- | --------- |
| Source files     | 200 lines     | 300 lines |
| React components | 150 lines     | 250 lines |
| Test files       | 200-300 lines | 500 lines |

The goal is to minimize reasoning surface area for agents, not to maximize file count.

- Most handwritten business-logic files should land in the `100-220` line range.
- `300` lines is the default hard cap for active source files because that is usually where edit accuracy, local search, and refactor safety start to degrade.
- Do not force cosmetic splits just to satisfy the counter. A cohesive file is better than three arbitrary shards.

## File Size Decision Rule

Use this order of preference:

1. Keep active source files under `300` lines when a real seam exists
2. Accept a documented exception when the file is cohesive and splitting would reduce clarity
3. Split aggressively only when it improves discoverability, dependency clarity, or testability

Treat `300` as a default cap, not a universal law.

### Allowed Exceptions

These can exceed `300` when splitting would be mostly mechanical:

- Static template data, large configuration maps, schema declarations, generated files, migrations
- Test files up to the existing `500` line ceiling
- Demo/example files that are low-churn and not part of core runtime behavior
- Tightly coupled classes or components with shared state, lifecycle, caches, event emitters, or transaction scope

For cohesive handwritten source, a practical ceiling of roughly `350-450` lines is acceptable only when:

- The file still represents one unit of reasoning
- The main exports are tightly coupled
- A split would create back-and-forth navigation without reducing complexity

Every source-file exception over `300` must be added to `.claude/file-size-exceptions.txt` with a short reason. If an exception is non-obvious, also document it in the plan, audit note, or session summary.

## Function and Component Limits

- **Functions**: Target 30 lines, max 50 lines
- **Stateless/presentational components**: Target 30 lines, max 40 lines
- **Stateful components**: Target 150 lines, max 250 lines

## Splitting Patterns

When a file is too large, extract into sibling files:

```
# Component splitting
BigComponent.tsx        → BigComponent.tsx (main render + state)
                        → BigComponentUtils.ts (helpers)
                        → BigComponentTypes.ts (interfaces/types)
                        → BigComponentParts.tsx (sub-components)

# Service splitting
big-service.ts          → big-service.ts (core operations)
                        → big-service-validation.ts (input validation)
                        → big-service-types.ts (interfaces)

# Test splitting
big.test.ts             → big-crud.test.ts
                        → big-validation.test.ts
                        → big-edge-cases.test.ts
```

## Split Signals

Split a file when one or more of these are true:

- It mixes multiple concerns such as data access, validation, orchestration, and formatting
- It has multiple exports that can be understood independently
- A reader must scroll through unrelated helpers to change one behavior
- The file has repeated setup branches that can be extracted into named helpers
- Tests would become easier to write after the extraction

Do not split when the only result is more files with the same mental model spread across them.

## Exports

- One primary export per file (default or named)
- Related helper exports are fine if they're tightly coupled
- Avoid barrel files (index.ts) that re-export more than 5-6 items
- Never create a barrel file over 50 lines

## Code Clarity

- Prefer explicit, verbose code over clever, compact code
- Use descriptive variable/function names (agents parse text, long names help)
- Each function/block should have unique identifiable markers (distinct names, not duplicated patterns)
- Avoid deep nesting (3+ levels of callbacks/conditionals) — extract into named functions

## YAGNI

Do not add unrequested features, abstractions, or utilities. Make the requested change. Nothing more.

Agents have a tendency to over-engineer. Every abstraction not requested is debt that has to be reasoned about on every future edit.

## When Modifying Existing Large Files

If you need to edit a file that already exceeds 300 lines:

1. Make the requested change
2. Decide whether the file is a real split candidate or a justified exception
3. If the change adds 20+ lines and a real seam exists, extract a logical chunk into a separate file in the same commit
4. If the file remains oversized because cohesion is higher value than fragmentation, add or update the matching entry in `.claude/file-size-exceptions.txt`
5. If the exception is non-obvious, record that explicitly in the plan, audit note, or session summary

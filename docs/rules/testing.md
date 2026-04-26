# Testing Rules

These rules apply to all projects using this baseline. They are non-negotiable — they exist because agents consistently write bad tests without them.

## The Core Rules

- **TDD: Write failing tests FIRST (red), then implement to pass (green). Non-negotiable.**
- Use Vitest for all unit and integration tests (unless the project already uses something else — don't change test frameworks without a compelling reason)
- Co-locate tests: `foo.ts` → `foo.test.ts` in the same directory
- Test files can be up to 500 lines (larger than source due to repetitive setup structure)
- Test both happy path AND error cases for every function
- **NEVER write tests that mirror implementation** — test BEHAVIOR, not IMPLEMENTATION
- Use descriptive test names: `it('returns error when input is missing required field')`
- NEVER use `it.skip` or `test.skip` — skipped tests are lies
- NEVER commit with failing tests

## Why TDD Matters for Agents

Without TDD, agents write "mirror tests" — tests that assert the same structure as the implementation. These tests pass trivially but catch nothing. A mirror test for `add(a, b) => a + b` would assert `add(2, 3) === 5` — it passes, but it doesn't test error cases, type coercion, or edge cases.

TDD forces you to define the expected behavior before writing code. This produces tests that actually catch regressions.

## Test File Organization

### Co-location (preferred)

```
src/
├── user-service.ts
├── user-service.test.ts     ← same directory
├── auth-validator.ts
└── auth-validator.test.ts   ← same directory
```

### Integration tests

For tests that require real infrastructure (DB, network), use a separate directory:

```
src/
└── __integration__/
    └── user-service.integration.test.ts
```

## What Good Tests Look Like

```typescript
// BAD — mirrors implementation
it('calls the database', async () => {
  const mockDb = { findUser: vi.fn().mockResolvedValue({ id: '1' }) }
  await getUser('1', mockDb)
  expect(mockDb.findUser).toHaveBeenCalledWith('1')
})

// GOOD — tests behavior
it('returns user data when user exists', async () => {
  const result = await getUser('existing-user-id', testDb)
  expect(result.id).toBe('existing-user-id')
  expect(result.email).toBeDefined()
})

it('returns error when user does not exist', async () => {
  const result = await getUser('nonexistent-id', testDb)
  expect(result).toBeNull() // or check error type if using Result pattern
})
```

## Running Tests

Standard Vitest commands (adapt to your project's package.json scripts):

```bash
npx vitest run          # Run all tests once
npx vitest watch        # Watch mode (TDD workflow)
npx vitest run --coverage  # With coverage
```

## Test Coverage Expectations

- Every public function in a source file should have a corresponding test
- Error paths are as important as happy paths
- Edge cases matter: empty arrays, null inputs, boundary values

The rule of thumb: if you can't write a test that fails before your implementation, you don't understand what you're building yet.

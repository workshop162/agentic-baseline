# /refactor-fix Command

Implement refactors from a refactor review, one finding at a time.

## Usage

`/refactor-fix [refactor-file]` — e.g., `/refactor-fix audits/refactor-src-2026-04-11.md`
`/refactor-fix next` — pick up the next pending finding from the most recent refactor review
`/refactor-fix all` — run through all pending findings continuously

## Process

For each finding with `Status: pending`:

1. **Read the finding** — understand what needs to change and why
2. **Implement the refactor** — make the code changes
3. **Run verification** — typecheck at minimum, plus whatever the finding specifies
4. **If verification passes**:
   - Update status: `Status: pending` → `Status: ✅ done (YYYY-MM-DD)`
   - Add a brief note of what changed
5. **If verification fails**:
   - Update status: `Status: pending` → `Status: ⚠️ attempted — [reason]`
   - In `next` mode: stop and ask the user
   - In `all` mode: log the issue and continue to next finding
6. **Commit** with message: `refactor: [finding ID] - [brief description]`

## Rules for `next` mode

- One finding at a time
- Stop after each fix and wait for confirmation

## Rules for `all` mode

- Work through all pending findings continuously
- Use subagents for independent refactors that don't touch the same files
- If a refactor is risky (marked high risk), skip it and log: `Status: ⏭️ skipped — high risk, needs manual review`
- Run `/compact` if context gets long
- At the end, show summary table of done/attempted/skipped

## General Rules

- **Always typecheck after each refactor** — refactors should never introduce type errors
- **Don't change behavior** — refactors improve code quality without changing what the code does
- **Respect the risk level** — low risk = just do it, medium = be careful, high = explain before touching
- **Keep commits atomic** — one finding per commit in `next` mode, group by area in `all` mode

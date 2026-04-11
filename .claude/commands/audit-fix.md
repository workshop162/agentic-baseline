# /audit-fix Command

Implement fixes from an audit report.

## Usage

`/audit-fix [audit-file]` — e.g., `/audit-fix audits/audit-src-2026-04-11.md`
`/audit-fix next` — pick up the next pending finding from the most recent audit
`/audit-fix all` — fix ALL pending findings continuously without stopping (see All Mode below)

## Process (Default — One at a Time)

For each finding with `Status: pending`:

1. **Read the finding** — understand the issue, file, and proposed fix
2. **Implement the fix** — make the code changes
3. **Run verification** — execute whatever the finding's Verification field says (typecheck, test, etc.)
4. **If verification passes**:
   - Update the finding's status in the audit file: `Status: pending` → `Status: ✅ fixed (YYYY-MM-DD)`
   - Add a brief note of what was changed
5. **If verification fails**:
   - Update status: `Status: pending` → `Status: ⚠️ attempted — [reason it failed]`
   - Do NOT move on — ask the user how to proceed
6. **Show summary** — what was changed, what was verified, what's next

## All Mode (`/audit-fix all`)

Continuously work through EVERY pending finding without stopping for user confirmation.

### All Mode Process

1. **Read the most recent audit file** (or the specified file)
2. **Collect all findings** with `Status: pending` or `Status: ⏳ deferred`, ordered by severity: Critical → High → Medium → Low
3. **Plan parallelization** — scan all pending findings and group ones that touch completely different files with no shared dependencies. These groups can be fixed simultaneously using subagents.
4. **For each finding (or parallel group), without pausing**:
   a. Read the finding — understand the issue, file, and proposed fix
   b. **If the finding can be parallelized** with other independent findings:
      - Launch subagents for each independent finding in the group
      - Each subagent implements the fix, runs verification, and reports back
      - Collect results from all subagents before updating statuses
   c. **If the finding must be sequential** (shares files with another pending fix):
      - Implement the fix directly
      - Run verification
   d. **If verification passes**:
      - Update status in audit file: `Status: pending` → `Status: ✅ fixed (YYYY-MM-DD)`
      - Add a brief note of what was changed
      - **Immediately move to the next finding**
   e. **If verification fails**:
      - Revert the changes for this finding (`git checkout -- <files>`)
      - Update status in audit file: `Status: pending` → `Status: ⚠️ attempted — [reason it failed]`
      - Add a note explaining what went wrong
      - **Keep going — move to the next finding**
5. **After each batch of fixes**, commit all successful fixes together:
   - Message: `fix: audit findings [IDs] — [brief summary]`
   - Do NOT commit one-at-a-time; batch by parallel group or every 5-8 sequential fixes
6. **Context management** — if the conversation is getting long (20+ findings processed), run `/compact` to compress context before continuing
7. **After all findings are processed**, print a summary table:

```
## Audit Fix Summary

| Finding | Severity | Status | Notes |
|---------|----------|--------|-------|
| C1 | Critical | ✅ fixed | Added auth check to handler |
| C2 | Critical | ⚠️ attempted | Requires env var that doesn't exist |
| H1 | High | ✅ fixed | Updated validation schema |
| M7 | Medium | ✅ fixed | Removed dead code |

**Fixed**: 12/30
**Attempted (failed verification)**: 2/30
**Still pending**: 16/30
```

### Parallelization Guidelines

Use subagents when findings are **fully independent**:

- Different files with no shared imports or exports
- No shared state (e.g., two findings in different modules)

Do NOT parallelize when:

- Two findings modify the same file
- One finding's fix depends on another's
- A finding modifies a shared utility used by other findings' files

When using subagents, give each one:

- The full finding text (issue, file, fix, verification)
- The project root path
- Instructions to: implement fix, run verification, report success/failure with details

### All Mode Rules

- **Never stop to ask** — the whole point is unattended continuous fixing
- **Never commit broken code** — if verification fails, revert changes for that finding before moving on
- **Batch commits** — group successful fixes into logical commits rather than one per finding
- **Don't skip verification** — always run the check, even in all mode
- **If a finding requires a database migration, external API change, or env var that doesn't exist** — mark as `⚠️ attempted — requires [thing]` and move on
- **If a finding's file doesn't exist or has changed significantly** — mark as `⚠️ attempted — file changed since audit` and move on
- **Respect severity order** — Critical first, Low last
- **Run /compact proactively** — compress after every 15-20 findings

## Default Mode Rules

- **One finding at a time** — implement, verify, update status, then stop and confirm with the user before moving to the next
- **Don't skip verification** — always run the check specified in the finding
- **Don't modify the finding's description** — only update the Status field and add notes below it
- **Respect severity order** — work Critical → High → Medium → Low unless the user says otherwise
- **If a fix would be risky or large**, explain the scope to the user before making changes
- **Commit after each fix** with message: `fix: [finding ID] - [brief description]`

## Rules (Both Modes)

- **Don't modify the finding's description** — only update the Status field and add notes below it
- **Respect severity order** — work Critical → High → Medium → Low unless the user says otherwise

## Example Status Updates

```markdown
- **Status**: ✅ fixed (2026-04-11)
- **Notes**: Added null check in getUser(), added test case in user-service.test.ts

- **Status**: ⚠️ attempted — fix requires database migration, needs discussion
- **Notes**: The column type change would affect 3 other services

- **Status**: ⏭️ skipped — user decided not to fix
- **Notes**: Accepted the tradeoff for now
```

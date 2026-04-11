# /bootstrap Command

Research and configure the best agentic-first tooling for this project. This is a one-time operation.

## Philosophy

You are not applying a preset. You are evaluating this specific project and making informed decisions about the best tooling for agentic development.

The decision records in `decisions/` show what similar projects chose — but those are inputs to your analysis, not templates to copy. Always research whether past choices are still the best option. Ecosystems change.

**Lead with a recommendation, not a menu.** Say "I recommend oxfmt because [specific reasons for this project]" — not "here are 5 options, pick one."

---

## Process

### Step 1: Read the project

Gather full context before forming any opinions.

- Read `package.json` (root + all workspace packages if monorepo)
- Read `README.md` and any existing docs
- Read all config files: `tsconfig.json`, `biome.json`, `.prettierrc`, `.eslintrc*`, `lefthook.yml`, `.husky/`
- Read existing `.claude/` if any
- Note: project type, structure (monorepo vs single), framework (Next.js, Vite, Node, library...), team size (solo vs team), existing tooling, test setup

Build a mental profile: *"This is a [type] project using [stack] with [existing tools], built by [solo/team]."*

### Step 2: Read the evaluation framework

Read `docs/AGENTIC-TOOLING-CRITERIA.md`.

Internalize what makes a tool agentic-first: speed, determinism, error message quality, zero-config, CLI-first, scriptable exit codes, fast feedback loop, native TypeScript support, composability.

### Step 3: Read similar decisions

Read the `decisions/` directory. Find records with tags matching this project's profile.

For each relevant decision record, note:
- What the project chose and why
- What was considered and rejected
- Any "What Worked / What Didn't" feedback

**Do not copy these decisions.** Use them as calibration. Ask: "Is this still the best choice? Does this project have constraints that make a different choice better?"

### Step 4: Research current best options

For each tooling category, evaluate what's best for THIS project:

#### Formatter
- Does the project already have one? If it's working well, keep it — don't churn.
- If not: evaluate oxfmt (speed-focused, sub-ms, TS/JS only), Biome (unified format+lint, zero-config, fast), Prettier (slow but widest ecosystem support).
- Key question for agentic fit: will this run in a PostToolUse hook without slowing the agent?

#### Linter
- Does the project already have one configured? Check if it's actually being run.
- If using Biome for format, use Biome for lint (same toolchain = simpler config).
- If using oxfmt, evaluate oxlint (same Oxc toolchain) vs Biome for lint.
- ESLint: only if the project has complex plugin requirements that other linters can't satisfy.
- Key question: are error messages specific enough for an agent to self-correct?

#### Test framework
- Does the project already use Jest, Vitest, or node:test? Don't switch without a compelling reason.
- If no framework: Vitest for TypeScript-first projects (fast, native ESM, minimal config).
- Key question: is there a watch mode for TDD? Are error messages clear enough to diagnose failures?

#### Git hooks
- Solo developer with PostToolUse hooks: PostToolUse hooks may be sufficient. Git hooks add value for pre-commit typechecking.
- Team project: use a hook runner (Lefthook preferred — fast, parallel, good DX; Husky is acceptable if already configured).
- What to run in pre-commit: typecheck at minimum, linter if fast enough.
- Key question: what's the CI setup? Hooks should mirror CI checks, not add new ones.

#### Build and package manager
- **Do not change these unless there is a compelling reason.** If the project uses pnpm, keep pnpm. If it uses tsc, keep tsc.
- Note what exists for the decision record, but don't churn.

### Step 5: Present recommendations

Show the user:

```
## Bootstrap Analysis: [Project Name]

### Project Profile
- Type: [e.g., TypeScript monorepo, Node library, Next.js app]
- Structure: [single package vs monorepo with X packages]
- Existing tooling: [what's already configured]
- Team: [solo / small team / team]

### What similar projects used
- [Project A]: [formatter], [linter], [test], [hooks]
- [Project B]: ...

### My recommendations

**Formatter**: [recommendation] — [specific reasoning for this project]
**Linter**: [recommendation] — [specific reasoning]
**Test framework**: [recommendation] — [specific reasoning]
**Git hooks**: [recommendation] — [specific reasoning]
**Build / package manager**: No changes recommended.

### Tradeoffs considered
[Any cases where the choice was close, what was weighed]

### What I'll configure
- [List of specific files/hooks that will be created or modified]
```

Wait for the user to confirm or redirect before proceeding.

### Step 6: Configure

After user confirmation:

**CLAUDE.md** — Generate from project analysis. Include:
- Project name and one-sentence description
- Architecture overview (packages, apps, key dependencies)
- Critical rules (YAGNI, guardrails, package boundaries if monorepo)
- Common commands (dev, test, typecheck, format, lint)
- Conventions (naming, commits, file size)
- Links to relevant docs

**`.claude/settings.json`** — Update with chosen hooks:
- Keep the universal file size hook (already present)
- Add formatter PostToolUse hook (run after every Edit/Write)
- Add linter PostToolUse hook if fast enough
- Add pre-commit typecheck PreToolUse hook (runs before `git commit`)
- Add project-specific boundary/pattern hooks if applicable

**`.claude/rules/`** — Copy universal files (already present):
- `coding-standards.md` — already there
- `testing.md` — already there
- Generate project-specific rules if needed (e.g., package boundaries, architecture constraints)

**`.claude/commands/`** — Already present (deep-audit, audit-fix, refactor, refactor-fix)

**`.claude/skills/`** — Already present (fix-issue, implement-feature, review)

**`.claude/agents/package-boundary-enforcer.md`** — If monorepo: populate with real package graph. If single package: remove the stub.

### Step 7: Record the decision

Create `decisions/[YYYY-MM-DD]-[project-slug].md`:

```markdown
---
project: [project-name]
date: [YYYY-MM-DD]
tags: [typescript, monorepo, pnpm, nextjs, ...] ← project characteristics
---

# [Project Name] — Bootstrap Decisions

## Project Profile
- Type: ...
- Structure: ...
- Size: ...
- UI: ...
- Framework: ...
- DB: ...

## Tooling Decisions

### Formatter: [choice]
**Why**: ...
**Considered**: ...
**Agentic fit**: Excellent / Good / Acceptable

### Linter: [choice]
**Why**: ...
**Considered**: ...
**Agentic fit**: ...

### Test Framework: [choice]
**Why**: ...
**Considered**: ...
**Agentic fit**: ...

### Git Hooks: [choice]
**Why**: ...
**Considered**: ...

### Build: [kept existing / changed to X]
**Why**: ...

## What Worked / What Didn't
(Leave empty — update after the project has been running a while)
```

### Step 8: Push decision to baseline

```bash
cd /tmp/agentic-baseline-update
git clone https://github.com/workshop162/agentic-baseline .
cp /path/to/project/decisions/[YYYY-MM-DD]-[project-slug].md decisions/
git add decisions/
git commit -m "decisions: add [project-name] bootstrap record"
git push
cd /path/to/project
rm -rf /tmp/agentic-baseline-update
```

### Step 9: Clean up

Remove bootstrap artifacts from the local project — they live in the baseline repo, not here:

```bash
rm -rf docs/           # Evaluation framework + philosophy lives in baseline repo
rm -rf decisions/      # Decision records live in baseline repo
rm .claude/commands/bootstrap.md  # Bootstrap is a one-time operation
```

The local `.claude/` is now clean: settings, rules, commands (audit + refactor), skills, agents — and nothing else.

---

## What to ask the user

Only ask when you genuinely can't decide:

- "No formatter configured. I'd recommend Biome based on [specific reasons for this project]. Want me to set it up, or do you prefer something else?"
- "No test framework yet. Want me to add Vitest, or skip for now?"
- "This is a team project. I'd add Lefthook for pre-commit hooks. Confirm?"

Do NOT ask about things with clear answers. Do NOT present a menu.

---

## Rules

- **Never change tooling the project is already using without a compelling reason.** Don't churn.
- **Never install packages without asking.** Show the `npm install` / `pnpm add` commands and wait for confirmation.
- **Never break existing CI.** If the project has GitHub Actions, read the workflow files before configuring hooks.
- **Always generate CLAUDE.md from the actual project** — don't use a template.
- **Always push the decision record** before cleaning up locally.

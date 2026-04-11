# Agentic Tooling Criteria

This is the evaluation framework the bootstrap agent uses when researching tooling for a new project.

**Not "use X" — but "here's how to evaluate whether X is the right choice."**

---

## What Makes a Tool Agentic-First?

| Criterion | Why It Matters | Example |
|-----------|---------------|---------|
| **Speed** | PostToolUse hooks run on every edit. Slow tools block the agent. | oxfmt (<1ms) vs Prettier (~200ms) |
| **Deterministic output** | Agents can't handle flaky formatting or linting. Same input must always produce same output. | Most formatters pass this. Some ESLint auto-fix rules don't. |
| **Clear, actionable error messages** | The agent reads the error and self-corrects. Vague errors cause loops. | "Line 42: unused import 'foo'" vs "found 3 issues" |
| **Zero/minimal config** | Less config = less for the agent to break. Opinionated defaults > configurable everything. | Biome (zero-config) vs ESLint (config maze) |
| **CLI-first** | Must work well from shell commands. GUI-only tools are useless for agents. | All modern tools pass this. |
| **Scriptable exit codes** | Hooks need `exit 0` (pass) vs `exit 1` (fail). Tools must signal clearly. | Most pass. Some tools exit 0 on warnings. |
| **Fast feedback loop** | Watch mode, incremental builds, fast test runs. TDD requires sub-second feedback. | Vitest (instant) vs Jest (cold start) |
| **Good TypeScript support** | First-class TS, not "also works with TS via plugin." | Vitest, Biome, oxlint all native. ESLint needs @typescript-eslint. |
| **Composable** | Can run as part of a pipeline. Doesn't demand exclusive control. | oxfmt + oxlint compose well. Some tools want to own format + lint + build. |

---

## Tooling Categories to Evaluate

For each new project, the bootstrap agent evaluates:

### 1. Formatter

What to evaluate:
- **Speed**: Will it work as a PostToolUse hook without slowing down every edit?
- **Determinism**: Does it produce consistent output? (All major formatters do.)
- **TS support**: Does it understand TypeScript syntax? (All major formatters do.)
- **Config burden**: How much config is needed to get it working?

Current options:
- **oxfmt** — Sub-millisecond, zero-config, TypeScript/JavaScript only. Excellent for PostToolUse. No other language support.
- **Biome** — Fast (<10ms), zero-config, handles TS/JS/JSON/CSS. Also lints — if you choose Biome for format, use it for lint too.
- **Prettier** — Slowest (~100-300ms), most ecosystem support, many plugin options. Acceptable for pre-commit; borderline for PostToolUse.
- **dprint** — Fast, plugin-based. Good for multi-language projects. More config than oxfmt.

When to choose what:
- TypeScript-only project, want fastest PostToolUse: oxfmt
- Want format + lint in one tool: Biome
- Project has non-TS files (MDX, other langs) and needs Prettier plugins: Prettier
- Don't change what's already working well

### 2. Linter

What to evaluate:
- **Error message quality**: Can an agent read the error and know exactly what to fix?
- **Speed**: Fast enough for PostToolUse or pre-commit?
- **TS support**: First-class, not via plugin?
- **Auto-fix reliability**: If it can fix, does it fix correctly? Some ESLint auto-fixes introduce bugs.

Current options:
- **oxlint** — Fast, complements oxfmt (same Oxc toolchain), good TS support. Doesn't have the full ESLint plugin ecosystem.
- **Biome** — If you're using Biome for format, use it for lint too. Zero additional config.
- **ESLint** — Most rules, most plugins (React, Next.js, testing-library, etc.). Slow. Config-heavy. Necessary when you need its ecosystem.
- **TypeScript compiler itself** — `tsc --noEmit` catches type errors, not style issues. Always run this regardless of linter.

When to choose what:
- Using oxfmt for format: pair with oxlint
- Using Biome for format: pair with Biome lint
- Project needs React/Next.js-specific rules, i18n rules, etc.: ESLint (with @typescript-eslint)
- Don't change a working linter setup

### 3. Test Framework

What to evaluate:
- **Speed**: TDD requires fast feedback. Sub-second test runs are ideal.
- **Watch mode**: Does it re-run affected tests on file change? Essential for TDD.
- **Error messages**: When a test fails, is the output clear enough to diagnose without reading the test?
- **ESM support**: TypeScript-first projects need native ESM support.
- **TypeScript support**: First-class, not via babel transform?

Current options:
- **Vitest** — Fast, native ESM, good TS support, familiar Jest-compatible API. Best default for new TypeScript projects.
- **Jest** — Widely known, large ecosystem, slower cold start, CJS-first (needs config for ESM). Use if the project already uses it.
- **node:test** — Built-in, no install, minimal features. OK for simple libraries, not for large projects.
- **Playwright** — For E2E and browser tests. Not a replacement for unit tests.

When to choose what:
- New project, no existing framework: Vitest
- Existing Jest project: keep Jest unless there's a compelling reason to migrate
- Need browser/E2E testing: add Playwright alongside unit test framework
- Don't change a working test setup

### 4. Git Hooks

What to evaluate:
- **Team size**: Solo developer with PostToolUse hooks may not need git hooks. Team projects need them.
- **CI integration**: Do hooks mirror what CI checks? They should.
- **Performance**: Pre-commit hooks must complete in a reasonable time (<60s).
- **Developer experience**: Hooks that are slow or noisy get disabled.

Current options:
- **PostToolUse hooks only** — For sole developers. Catches issues at edit time. No separate hook runner needed.
- **Lefthook** — Fast (parallel execution), good DX, YAML config. Preferred for team projects.
- **Husky** — Most widely known, simple setup. Slightly more overhead than Lefthook.
- **simple-git-hooks** — Minimal, low-overhead. Good for small projects.

What to run in pre-commit:
- `tsc --noEmit` (typecheck) — Always. Blocks broken TS from reaching the repo.
- Linter — If fast enough (oxlint/Biome: yes. ESLint: marginal).
- Tests — Only for fast test suites. Running 5 minutes of tests pre-commit breaks flow.

When to choose what:
- Solo developer: PostToolUse hooks + pre-commit typecheck
- Small team: Lefthook with typecheck + linter
- Existing Husky setup: keep it, add checks

### 5. Build

**Default: Don't change what's configured.** If the project uses `tsc`, keep it. If it uses `tsup`, keep it. If it uses Vite, keep it.

Change only if:
- The current build is clearly wrong for the project type
- The user explicitly asks to reconsider build tooling

### 6. Package Manager

**Default: Don't change what's configured.** If the project uses pnpm, keep pnpm. If it uses npm, keep npm.

Change only if the project is just starting and has no preference.

---

## How to Evaluate Tools You're Unsure About

Ask:

1. **How fast is it?** Under 10ms is excellent for PostToolUse. Under 100ms is acceptable.
2. **What does a failure message look like?** Read the docs or run it on a test file with a known error. Is the output actionable?
3. **How much config does it need?** Zero or minimal is best. Extensive config is a maintenance burden.
4. **Does it exit 1 on errors?** Run `echo $?` after a failing run. Some tools exit 0 even on errors.
5. **When was the last release?** A tool with a release in the last 6 months is actively maintained. Older may still be fine, but check the issue tracker.

---

## Anti-patterns to Avoid

- **Churn**: Don't switch tools because a different project used something else. Switch only if the current tool is genuinely worse for agentic development.
- **Menus**: Don't present 5 options and ask the user to pick. Research and make a recommendation.
- **Assumed defaults**: Don't configure a tool and then find out CI was already running a different version.
- **Installing unused tooling**: Don't add oxlint if the project doesn't plan to use it. Unused dev dependencies are noise.
- **Ignoring existing config**: Read `.prettierrc`, `biome.json`, `.eslintrc` before assuming nothing is configured.

# Changelog — Evolution of Agentic-First Development

This tracks how the patterns in this baseline were discovered and refined across real projects.

---

## v7 — AGENTS.md Adoption (2026-04-26)

**What changed**: The baseline (and every project bootstrapped from it) now uses [`AGENTS.md`](https://agents.md) as the canonical agent-instruction file instead of `CLAUDE.md`. The agents.md format is an open standard stewarded by the Linux Foundation's Agentic AI Foundation, supported natively by Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, Windsurf, Aider, Zed, Warp, RooCode, and others.

**Why**: A growing fraction of agentic-first work happens across multiple coding agents — Claude Code for some workstreams, Codex for others, Cursor for IDE-native work. Maintaining `CLAUDE.md` forced multi-agent teams to either duplicate instructions across `CLAUDE.md` + `.cursorrules` + `copilot-instructions.md` (drift inevitable) or write tool-specific overlays. The agents.md standard solves this with one canonical file every modern agent loads automatically. Claude Code falls back to `AGENTS.md` when no `CLAUDE.md` is present, so adopting the standard adds zero friction for Claude-only workflows while making the project portable across the rest of the ecosystem.

**Key changes**:
- `CLAUDE.md` → `AGENTS.md` (universal canonical file)
- `.claude/rules/` → `docs/rules/` (universal coding/testing rules are not Claude-specific; the directory name was a misnomer)
- The `.claude/` directory now contains only Claude Code-specific harness pieces: `settings.json`, `commands/`, `skills/`, `agents/`. Other agents read these as plain markdown when relevant.
- `bootstrap.md` updated to generate `AGENTS.md` (canonical) + optionally a thin `CLAUDE.md` only when the project has Claude-specific overlays
- New pattern documented: per-package `AGENTS.md` files for monorepo overlays (the agents.md standard supports nested files; closest one wins)

**Migration path for downstream projects already bootstrapped from v6 or earlier**:
1. `git mv CLAUDE.md AGENTS.md`
2. `git mv .claude/rules docs/rules`
3. Update internal references from `CLAUDE.md` → `AGENTS.md` and `.claude/rules/` → `docs/rules/`
4. Optional: add a thin `CLAUDE.md` if you have Claude-specific overlays (start with `@AGENTS.md` to inline the universal file)

**What did not change**: All universal patterns (file size limits, TDD, audit-then-fix, file-size hook, package boundary enforcement) carry over unchanged. The mechanical safety net (Lefthook + biome + typecheck + file-size hook) is still the real backstop.

---

## v6 — Learning Baseline (2026-04-11)

**What changed**: Extracted the `decisions/` knowledge base from the history embedded in this file. Every bootstrap now pushes a structured decision record back to the baseline. Future bootstraps read these records before making recommendations.

**Why**: Two projects isn't enough data to make rules. But with a growing knowledge base, the bootstrap agent can see patterns: "projects with this profile consistently chose X, and here's what worked." The agent still researches and decides independently — but now it has institutional memory.

**Key additions**:
- `decisions/` directory with README
- Seeded with GetPumped v3 and AgentiCNC records
- Bootstrap command restructured around research-first workflow
- `docs/AGENTIC-TOOLING-CRITERIA.md` — the evaluation framework the bootstrap agent uses

---

## v5 — Self-Configuring Baseline (2026-04-11)

**What changed**: The bootstrap command replaced static templates. Instead of "copy this CLAUDE.md template and fill in the blanks," the bootstrap agent reads the project, consults the knowledge base, and generates configuration from scratch.

**Why**: Templates drift from reality. A template created for a Next.js monorepo is wrong for a Node library. The bootstrap agent can analyze the actual project and make better decisions than any generic template.

**Key additions**:
- `.claude/commands/bootstrap.md` — the research + configure + record workflow
- File size hook as the only universal hook (formatter/linter hooks added by bootstrap)
- Package boundary enforcer as a stub (bootstrap populates with real graph)

---

## v4 — Extracted Universal Baseline (2026-04-11)

**What changed**: Extracted universal patterns from GetPumped v3 and AgentiCNC into this standalone repo. Removed all project-specific references.

**Why**: Two independent projects arrived at similar conventions (200/300 file limits, TDD enforcement, audit→fix workflow, boundary enforcers). The patterns that appeared in both projects became candidates for universal rules.

**Key universal patterns extracted**:
- `coding-standards.md` — 200/300 limits, split signals, cohesion exceptions
- `testing.md` — TDD, co-location, no `it.skip`, behavior not implementation
- `deep-audit.md` / `audit-fix.md` — audit format with severity levels, separate audit and fix steps
- `fix-issue/SKILL.md` / `implement-feature/SKILL.md` / `review/SKILL.md`
- File size PostToolUse hook

---

## v3 — AgentiCNC (2026-02-14 to 2026-04-10)

**Project**: AgentiCNC — greenfield millwork design platform replacing Cabinet Vision.

**What this project validated**:
- TDD from day 1 (greenfield) works much better than retrofitting TDD
- Biome as unified format+lint tool: zero-config, fast, excellent DX
- File size hard ceiling from day 1: caught issues early before they became load-bearing
- PostToolUse hooks alone (no git hooks) sufficient for sole developer
- Package boundary enforcer as a read-only agent: effective at catching violations early
- `neverthrow` Result types: effective, but requires discipline (the throw check hook helps)
- Co-located tests (`foo.test.ts` next to `foo.ts`): better discoverability than `__tests__/` directories

**What didn't work**:
- Nothing notable — greenfield projects allow you to set conventions before technical debt accumulates

**Key additions vs v2**:
- Boundary enforcer as read-only agent (v2 used static rules; v3 made it an agent for better reasoning)
- PostToolUse hook for missing test files (warns when a `.ts` file has no `.test.ts`)
- PostToolUse hook for forbidden patterns (FinalizationRegistry, Y.Map, throw statements)
- Biome as the format+lint choice (vs oxfmt+oxlint in GetPumped)

---

## v2 — GetPumped v3 (2025-11-01 to 2026-04-10)

**Project**: GetPumped v3 / PumpTrack — waste management SaaS (Next.js + Expo monorepo).

**What this project discovered**:
- Soft warnings are ignored; hard-fail at 300 lines is necessary
- Separate audit and fix steps prevent scope creep and regressions
- PostToolUse hooks for formatting (oxfmt) are invisible — they just work
- Pre-commit typecheck catches the most common agent mistake (committing broken TS)
- Lefthook (vs Husky) is worth the switch for parallel hook execution
- Package boundary violations accumulate if not caught early
- "Known Oversized Files" section in coding-standards.md doesn't work — agents add exceptions without thinking

**What didn't work**:
- Soft warning on file size: ignored consistently
- Audit-and-fix-simultaneously: caused regressions when multiple fixes conflicted
- Rules listing specific known-bad files: agents would add new files to the list rather than fixing them

**Key additions vs v1**:
- `.claude/rules/` directory (moved from inline CLAUDE.md)
- `.claude/settings.json` with PostToolUse hooks
- Hard-fail at 300 (replaced soft warning)
- `/deep-audit` and `/audit-fix` commands
- Lefthook for pre-commit hooks
- oxfmt + oxlint (replaced Prettier/ESLint)

---

## v1 — GetPumped v2 (2025-08-01 to 2025-10-31)

**Project**: GetPumped v2 — first project to use a structured CLAUDE.md.

**What was discovered**:
- Agents need explicit file size limits or they create unbounded files
- Agents need explicit YAGNI rules or they over-engineer
- Pre-commit hooks help but agents find ways around them
- A CLAUDE.md with clear architecture rules significantly reduces wrong-package imports
- Test coverage is the first thing agents skip when time-pressured

**What was present**:
- Manual CLAUDE.md with project description, architecture, and conventions
- Basic file size guidance (200 lines, no enforcement)
- Conventional commit messages
- Minimal testing rules

**What was missing**:
- File size enforcement (PostToolUse hooks)
- Separation of universal vs project-specific rules
- Audit commands
- Skills/workflows
- Any knowledge base for future projects

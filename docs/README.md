# Agentic Baseline — Setup Guide

## What is this?

`workshop162/agentic-baseline` is a bootstrap framework for TypeScript projects developed with AI agents. It provides:

1. **Universal rules** — file size limits, testing conventions that work for any TypeScript project
2. **Universal commands** — audit, refactor, and fix workflows
3. **Universal skills** — TDD-first feature and bug workflows
4. **A bootstrap command** — researches and configures the best agentic-first tooling for your specific project
5. **A decision knowledge base** — records what tooling was chosen for each project and why, so future bootstraps can learn from real experience

## Quick Start

### For a new project

```bash
# 1. Clone the baseline into a temporary directory
git clone https://github.com/workshop162/agentic-baseline /tmp/agentic-baseline

# 2. Copy the universal pieces into your project
cp -r /tmp/agentic-baseline/.claude/ /path/to/your-project/.claude/
cp -r /tmp/agentic-baseline/docs/rules/ /path/to/your-project/docs/rules/

# 3. Open Claude Code in your project
cd /path/to/your-project
claude

# 4. Run the bootstrap command
/bootstrap
```

The bootstrap agent will:
- Read your project (package.json, README, existing config)
- Read the evaluation framework (docs/AGENTIC-TOOLING-CRITERIA.md)
- Read decision records from similar projects
- Research current tooling options
- Present recommendations with reasoning
- Configure your `.claude/` after your confirmation
- Push a decision record back to this repo
- Clean up — removing docs/, decisions/, and bootstrap.md from your project

### What you'll end up with

After bootstrap, your project will contain:

```
your-project/
├── AGENTS.md                            # Generated — canonical agent-instruction file
├── docs/
│   └── rules/
│       ├── coding-standards.md          # Universal file/function size limits
│       ├── testing.md                   # Universal TDD rules
│       └── [project-specific].md        # Generated if needed (architecture, boundaries)
└── .claude/
    ├── settings.json                    # File size hook + your chosen formatter/linter/pre-commit hooks
    ├── commands/
    │   ├── deep-audit.md
    │   ├── audit-fix.md
    │   ├── refactor.md
    │   └── refactor-fix.md
    ├── agents/
    │   └── package-boundary-enforcer.md # Populated if monorepo, removed if not
    └── skills/
        ├── fix-issue/SKILL.md
        ├── implement-feature/SKILL.md
        └── review/SKILL.md
```

No `bootstrap.md`, no baseline `decisions/` or `docs/` (those live in the baseline repo). Clean.

`AGENTS.md` is what every coding agent (Claude Code, Codex, Cursor, Copilot, Gemini CLI, Windsurf, Aider, Zed, Warp, RooCode) loads automatically per the [agents.md](https://agents.md) standard. Bootstrap only generates a separate `CLAUDE.md` if the project has Claude-specific overlays — most projects skip it.

## What the baseline does NOT do

- It does not prescribe a stack
- It does not tell you to use oxfmt, Biome, Vitest, or any specific tool
- It does not apply presets based on detected dependencies
- It does not assume your project looks like any other project

The bootstrap agent evaluates your specific project and recommends tooling based on research and past decisions. The goal is informed configuration, not pattern-matching.

## Files in this repo

| Path | Purpose |
|------|---------|
| `docs/rules/coding-standards.md` | Universal file/function size limits |
| `docs/rules/testing.md` | Universal TDD + Vitest rules |
| `.claude/settings.json` | Minimal universal hook (file size only) |
| `.claude/commands/bootstrap.md` | The research + configure command |
| `.claude/commands/deep-audit.md` | Comprehensive code audit |
| `.claude/commands/audit-fix.md` | Fix audit findings (one-at-a-time or continuous) |
| `.claude/commands/refactor.md` | Code quality review |
| `.claude/commands/refactor-fix.md` | Fix refactor findings |
| `.claude/agents/package-boundary-enforcer.md` | Stub — bootstrap populates with real graph |
| `.claude/skills/fix-issue/SKILL.md` | Debug + fix workflow |
| `.claude/skills/implement-feature/SKILL.md` | TDD feature implementation |
| `.claude/skills/review/SKILL.md` | Quality review checklist |
| `decisions/` | Knowledge base — grows as bootstrap runs on projects |
| `docs/PHILOSOPHY.md` | Why each pattern exists |
| `docs/AGENTIC-TOOLING-CRITERIA.md` | Tool evaluation framework |
| `docs/PATTERNS.md` | Universal vs project-specific catalog |
| `docs/CHANGELOG.md` | Evolution history |

## Contributing

If you've used bootstrap on a new project and want to improve the baseline:

- Add feedback to your project's decision record ("What Worked / What Didn't")
- Open a PR to improve universal rules if you've found a better pattern
- Improve the evaluation framework if you've identified new agentic-first criteria

The baseline gets smarter as more projects use it.

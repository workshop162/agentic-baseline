# Agentic Baseline — AGENTS.md

> This is the canonical agent-instruction file for this repo. Loaded automatically by every major coding agent (Claude Code, OpenAI Codex, Cursor, GitHub Copilot, Gemini CLI, Windsurf, Aider, Zed, Warp, RooCode) per the [agents.md](https://agents.md) standard. Claude Code falls back to this file when no `CLAUDE.md` is present.

This is the `workshop162/agentic-baseline` repository — a **bootstrap framework and knowledge base** for agentic-first TypeScript development.

## What this repo is

**A framework for thinking, not a set of presets.**

The baseline ships universal rules, commands, and skills that work for any TypeScript project. The most important piece is the bootstrap command — it researches the best agentic-first tooling for each specific project by reading the project, reading the evaluation framework, consulting the decisions knowledge base, and making an informed recommendation.

## What this repo is NOT

This is not a template to copy-paste. It does not prescribe a stack. It does not tell you to use oxfmt or Biome or Vitest. It gives the agent a framework for evaluating those choices and recording what was decided and why.

## How to use it

### For a new project

1. Clone this repo into a temporary directory next to your project:
   ```bash
   git clone https://github.com/workshop162/agentic-baseline /tmp/agentic-baseline
   ```
2. Copy the universal pieces into your project:
   ```bash
   cp -r /tmp/agentic-baseline/.claude/ /path/to/your-project/.claude/
   cp -r /tmp/agentic-baseline/docs/rules/ /path/to/your-project/docs/rules/
   ```
3. Run `/bootstrap` in Claude Code (or any agent — the command itself is just a markdown spec readable by all)
4. The bootstrap agent will:
   - Read your project (package.json, README, existing config)
   - Read the evaluation framework (`docs/AGENTIC-TOOLING-CRITERIA.md`)
   - Read similar decisions from `decisions/`
   - Research current options
   - Present recommendations with reasoning
   - Generate `AGENTS.md` (the canonical project-instruction file) — Claude Code, Codex, and every other agents.md-compatible tool will pick it up automatically. **Only generate a thin `CLAUDE.md` if the project has Claude-specific overlays** (rare — auto-memory pointers, skill-only directives). Most projects will not need one.
   - Configure your `.claude/` (settings + hooks) after your approval
   - Push a decision record back to this repo
   - Clean up — removing docs/, decisions/, and the bootstrap command from your project

### For this repo itself

If you're working in this repo (adding decision records, improving the evaluation framework, updating universal rules):

- `decisions/` — add records after running bootstrap on a new project
- `docs/` — improve the framework, philosophy, patterns
- `docs/rules/` — update universal coding standards or testing rules
- `.claude/commands/` — improve audit, refactor, or bootstrap commands (Claude Code-flavored, but markdown specs that any agent can read and follow)
- `.claude/skills/` — improve TDD workflows (same)

## Guardrails

Do not bypass, disable, or weaken guardrails. If a hook fails, the problem is the code, not the hook.

Do not modify `.claude/settings.json` unless explicitly doing tooling work.

## Repository structure

```
agentic-baseline/
├── AGENTS.md                              # This file (canonical for all agents)
├── docs/
│   ├── README.md                          # Setup guide
│   ├── PHILOSOPHY.md                      # Why each pattern exists
│   ├── AGENTIC-TOOLING-CRITERIA.md        # Tool evaluation framework
│   ├── PATTERNS.md                        # Universal vs project-specific
│   ├── CHANGELOG.md                       # Evolution history
│   └── rules/
│       ├── coding-standards.md            # Universal file/function limits
│       └── testing.md                     # Universal TDD + Vitest rules
├── .claude/                               # Claude Code-specific harness pieces
│   ├── settings.json                      # Minimal: file size hook only
│   ├── commands/
│   │   ├── bootstrap.md                   # Research + configure + record
│   │   ├── deep-audit.md                  # Comprehensive code audit
│   │   ├── audit-fix.md                   # Fix audit findings
│   │   ├── refactor.md                    # Code quality review
│   │   └── refactor-fix.md                # Fix refactor findings
│   ├── agents/
│   │   └── package-boundary-enforcer.md   # Stub — bootstrap populates
│   └── skills/
│       ├── fix-issue/SKILL.md             # Debug + fix workflow
│       ├── implement-feature/SKILL.md     # TDD feature implementation
│       └── review/SKILL.md                # Quality review checklist
└── decisions/                             # Knowledge base — grows over time
    ├── README.md
    └── seed-*.md                          # Seeded from existing projects
```

### Why universal rules live in `docs/rules/`

`coding-standards.md` and `testing.md` are project-universal — file size limits and TDD apply regardless of which agent is editing the code. Their previous home in `.claude/rules/` was a misnomer; they're not Claude-specific. They moved to `docs/rules/` in v7 of this baseline so that when Codex, Cursor, or any other agent reads `AGENTS.md` and follows a path reference, the path doesn't suggest "this is for Claude only."

The Claude Code-specific harness — `commands/`, `skills/`, `agents/`, `settings.json` — stays under `.claude/` because those directories are how Claude Code's loader discovers them. Other agents read their content as plain markdown when needed but don't auto-load them.

## Multi-agent compatibility

This repo's universal patterns work with every major coding agent in 2026:

| Agent | Loads `AGENTS.md` | Reads `docs/rules/` via paths | Auto-loads `.claude/{commands,skills,agents}` |
|---|---|---|---|
| Claude Code | yes (via fallback when no `CLAUDE.md`, or directly) | yes | yes |
| OpenAI Codex | yes (native) | yes | no — readable as plain markdown |
| Cursor | yes (native) | yes | no |
| GitHub Copilot | yes (native) | yes | no |
| Gemini CLI / Windsurf / Aider / Zed / Warp / RooCode | yes (native) | yes | no |

The mechanical safety net (Lefthook + biome + typecheck + file-size hook configured by bootstrap) catches violations regardless of which agent wrote the code.

# Agentic Baseline

This is the `workshop162/agentic-baseline` repository. It is a **bootstrap framework and knowledge base** for agentic-first TypeScript development.

## What this repo is

**A framework for thinking, not a set of presets.**

The baseline ships universal rules, commands, and skills that work for any TypeScript project. But the most important piece is the bootstrap command — it researches the best agentic-first tooling for each specific project by reading the project, reading the evaluation framework, consulting the decisions knowledge base, and making an informed recommendation.

## What this repo is NOT

This is not a template to copy-paste. It does not prescribe a stack. It does not tell you to use oxfmt or Biome or Vitest. It gives the agent a framework for evaluating those choices and recording what was decided and why.

## How to use it

### For a new project

1. Clone this repo into a temporary directory inside your project:
   ```bash
   git clone https://github.com/workshop162/agentic-baseline .baseline-tmp
   ```
2. Copy the `.claude/` directory into your project root
3. Run `/bootstrap` in Claude Code
4. The bootstrap agent will:
   - Read your project (package.json, README, existing config)
   - Read the evaluation framework (`docs/AGENTIC-TOOLING-CRITERIA.md`)
   - Read similar decisions from `decisions/`
   - Research current options
   - Present recommendations with reasoning
   - Configure your `.claude/` after your approval
   - Push a decision record back to this repo
   - Clean up — removing docs/, decisions/, and the bootstrap command from your project

### For this repo itself

If you're working in this repo (adding decision records, improving the evaluation framework, updating universal rules):

- `decisions/` — add records after running bootstrap on a new project
- `docs/` — improve the framework, philosophy, patterns
- `.claude/rules/` — update universal coding standards or testing rules
- `.claude/commands/` — improve audit, refactor, or bootstrap commands
- `.claude/skills/` — improve TDD workflows

## Guardrails

Do not bypass, disable, or weaken guardrails. If a hook fails, the problem is the code, not the hook.

Do not modify `.claude/settings.json` unless explicitly doing tooling work.

## Repository structure

```
agentic-baseline/
├── CLAUDE.md                              # This file
├── .claude/
│   ├── settings.json                      # Minimal: file size hook only
│   ├── rules/
│   │   ├── coding-standards.md            # Universal file/function limits
│   │   └── testing.md                     # Universal TDD + Vitest rules
│   ├── commands/
│   │   ├── bootstrap.md                   # Research + configure + record
│   │   ├── deep-audit.md                  # Comprehensive code audit
│   │   ├── audit-fix.md                   # Fix audit findings
│   │   ├── refactor.md                    # Code quality review
│   │   └── refactor-fix.md               # Fix refactor findings
│   ├── agents/
│   │   └── package-boundary-enforcer.md   # Stub — bootstrap populates
│   └── skills/
│       ├── fix-issue/SKILL.md             # Debug + fix workflow
│       ├── implement-feature/SKILL.md     # TDD feature implementation
│       └── review/SKILL.md                # Quality review checklist
├── decisions/                             # Knowledge base — grows over time
│   ├── README.md
│   └── seed-*.md                          # Seeded from existing projects
└── docs/
    ├── README.md                          # Setup guide
    ├── PHILOSOPHY.md                      # Why each pattern exists
    ├── AGENTIC-TOOLING-CRITERIA.md        # Tool evaluation framework
    ├── PATTERNS.md                        # Universal vs project-specific
    └── CHANGELOG.md                       # Evolution history
```

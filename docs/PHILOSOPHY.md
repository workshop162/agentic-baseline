# Philosophy — Why Each Pattern Exists

This document explains the reasoning behind every universal convention in the baseline. Understanding the "why" is what turns rules into judgment.

---

## Why 200/300 line limits

**The problem**: AI agents write longer files than humans do. Left unconstrained, an agent will add functions to an existing file rather than creating a new one, because that's the path of least resistance. Over time, files grow to 500, 800, 1000+ lines.

**Why this matters**: Agent edit accuracy degrades as files grow. When an agent reads a 600-line file to make a 5-line change, it has to hold the entire file in context. Larger context = more reasoning = higher probability of mistakes, missed edge cases, and unintended side effects.

**Why 200 soft / 300 hard**: These numbers come from empirical observation across multiple projects. Below 200 lines, most files are easy to reason about. Between 200-300, it's worth checking for a split. Above 300, accuracy starts to noticeably decline. At 400+, it's always worth splitting if a real seam exists.

**The "cohesion over fragmentation" exception**: The rule is not "always split at 300." It's "split when there's a real seam — a logical boundary where the two halves are more independent than they are coupled." Splitting a tightly-coupled module into two files that import each other constantly is worse than keeping it as one 350-line file.

---

## Why YAGNI (You Aren't Gonna Need It)

**The problem**: Agents over-engineer. Given "add a user lookup function," an agent might add the function, create a shared utility for future lookup use cases, add a caching layer "for performance," and extract an interface for testability — none of which was asked for.

**The cost**: Every abstraction the agent adds is something future agents have to understand, route around, or maintain. Over-engineered code is harder to change than simple code.

**The rule**: Make the requested change. Nothing more. If the user wants a caching layer, they'll ask for it. If they don't ask, assume they don't need it.

---

## Why PostToolUse hooks (not just pre-commit hooks)

**The problem**: Pre-commit hooks catch issues at commit time — which might be 30 minutes or 30 edits after the mistake was made. The agent has moved on. Fixing it now means re-reading context you've forgotten.

**PostToolUse advantage**: Runs after every Edit or Write operation. The agent gets feedback on the file it just modified, while it still has full context. A formatter hook running after every edit means the code is always formatted — no "fix formatting" commit needed. A file size hook after every edit means the agent can't accidentally create a 400-line file without noticing.

**The trade-off**: PostToolUse hooks must be fast. A hook that takes 2 seconds runs on every edit — over 100 edits, that's 3+ minutes of waiting. This is why formatter choice matters: oxfmt (sub-ms) is a better PostToolUse hook candidate than Prettier (~200ms).

---

## Why hard-fail at 300 (not just a warning)

**The problem**: Soft warnings get ignored. If the agent sees "WARNING: this file has 250 lines," it notes it and moves on. If the CI is green, the code ships.

**What hard-fail does**: `exit 1` from a PostToolUse hook blocks the current operation and forces the agent to address the issue immediately. The agent can't move on to the next task until the file is under the limit.

**The exception path**: The exception mechanism (`.claude/file-size-exceptions.txt`) exists for legitimate cases. Template data, generated files, tightly-coupled classes — these can exceed 300 with a documented reason. The hard-fail isn't "you can never have a 300-line file," it's "you can't have one without acknowledging the decision."

---

## Why read-only boundary enforcer agents

**The problem**: In a monorepo, agents "fix" violations by the most expedient path — which is often adding an import rather than creating the right abstraction. An agent working in `packages/ui` might import from `packages/api` because "it's easier than creating a shared interface."

**What the enforcer does**: Runs as a read-only agent that scans for boundary violations. Read-only is critical — the enforcer's job is to report, not to "fix." If it could edit, it might make the wrong architectural decision.

**The result**: Boundary violations are surfaced immediately, before they become load-bearing code. The human decides how to fix them.

---

## Why TDD skills (explicit workflow, not just rules)

**The problem**: Agents write mirror-tests without TDD enforcement. A mirror test for `add(a, b) => a + b` asserts `add(2, 3) === 5`. It passes. It tells you nothing. If you change `add` to return `a * b`, the test still passes because you forgot to update it.

**What TDD forces**: Writing a failing test first forces you to define expected behavior independent of implementation. If the test passes before you write implementation code, you've written the wrong test.

**Why this has to be a skill**: A rule in `.claude/rules/testing.md` says "write failing tests first." A skill in `.claude/skills/implement-feature/SKILL.md` walks through the entire TDD cycle with explicit verification steps. The skill makes it harder to skip the failing-test step accidentally.

---

## Why pre-commit typecheck

**The problem**: Agents commit TypeScript code with type errors. Without a pre-commit check, broken code reaches the repo. Other agents pull the broken code and spend time debugging type errors they didn't introduce.

**What it does**: Blocks `git commit` if typecheck fails. The agent must fix type errors before committing.

**Why not run it as a PostToolUse hook**: Typecheck is slower than formatting (5-30 seconds for a typical project). Running it after every edit would make every edit noticeably slower. Pre-commit is the right balance — it runs once per commit, at the point where correctness matters most.

---

## Why audit then fix as separate steps (not audit-and-fix)

**The problem**: When an agent audits and fixes in the same operation, it introduces regressions. The agent makes 10 fixes simultaneously, some of which conflict. Or it finds a fix that requires a larger refactor and scopes-creeps into a 2-hour rewrite.

**The two-step approach**: `/deep-audit` produces a prioritized findings file. `/audit-fix next` implements one finding at a time, runs verification, and stops. Each fix is tested independently before the next one starts.

**Why this matters**: An audit finding might be wrong. The proposed fix might be wrong. Running verification after each fix catches problems at the smallest possible scope. If fix #3 breaks tests, you know it was fix #3, not "something in the last 10 changes."

---

## Why the guardrails rule ("don't bypass, disable, or weaken guardrails")

**The problem**: When a guardrail fails, the path of least resistance is to disable the guardrail. An agent blocked by a linting rule might comment out the rule. An agent failing a typecheck might add `// @ts-ignore`. An agent failing a boundary check might add a `// NOSONAR` comment.

**The rule**: Guardrails exist because they've caught real problems. If a guardrail fails, assume the problem is your code, not the guardrail. Fix the code.

**The exception**: Sometimes guardrails are wrong — too strict, or failing on a legitimate pattern. The right response is to discuss and update the guardrail explicitly, not bypass it silently. This keeps the guardrail changes visible and intentional.

---

## Why the knowledge base (decisions/) grows over time

**The problem with static templates**: A template says "use Biome." But Biome might be the wrong choice for a project with complex ESLint plugins, or for a project where the team already knows Prettier. A template can't capture that nuance.

**What decision records add**: Each record captures not just what was chosen, but *for what kind of project*, and *what the tradeoffs were*. Future bootstraps can see "projects with this profile chose X, and here's why. Is that still right for you?"

**The "consider but never blindly copy" rule**: Seeing "codesight used oxfmt" is input, not a prescription. The bootstrap agent evaluates whether oxfmt is still the best choice, whether anything has changed, and whether this new project has different constraints. The knowledge base informs decisions — it doesn't make them.

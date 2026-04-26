---
project: vex-override-sim
date: 2026-04-25
tags: [typescript, monorepo, pnpm, vite, react, hono, trpc, colyseus, babylon, havok, rapier3d, onnx, python, uv, pytorch, ppo, rl, multiplayer, simulation, browser-first]
---

# VEX Override Simulator — Bootstrap Decisions

Open-source dual-purpose simulator for the VEX V5 Robotics Competition 2026–2027 game *Override*. One TypeScript monorepo serves a multiplayer browser game (1–4 players, human/AI mix) and a headless RL training environment (PPO self-play + ONNX policies hot-loaded back into the game).

The working plan is `nimbalyst-local/plans/vex-override-simulator-v2.md` in the project. It locks several stack choices in advance — bootstrap respects those and only re-evaluates the unlocked categories (formatter, linter, hooks).

## Project Profile

- **Type**: TypeScript monorepo with a Python sidecar (training/)
- **Structure**: pnpm workspaces — `apps/web`, `apps/server`, `packages/{schemas, sim-core, physics-babylon-havok, physics-rapier, render-2d, render-babylon, policy-runtime, bots, rl-bridge}`, `training/`
- **Size**: solo developer (Brian Sweet) for v1; OSS contributions expected after v1 release
- **UI**: Vite SPA + React + TanStack Router (browser-first; desktop wrapper deferred to v1.1)
- **Server**: Hono + tRPC (fetch adapter) + Colyseus 0.16+ in one Node 22 process for v1
- **Engine**: Babylon.js 9 + @babylonjs/havok primary; @dimforge/rapier3d-deterministic warm fallback; Godot 4.6 third-tier fallback
- **Inference**: onnxruntime-node + onnxruntime-web (exact-pinned across Python/Node/Web)
- **DB**: Drizzle + SQLite (Postgres path open if v2 demands)
- **Schemas**: Zod (versioned across replays and policies)
- **Training**: Python 3.12, `uv`, PyTorch (CPU vs MPS benchmarked, not assumed), CleanRL or Sample Factory PPO

The plan (§3.1) locks a **contracts-first** architecture: `packages/sim-core` is renderer-agnostic and framework-agnostic. Renderers, networking, training, and inference are adapters around versioned Zod schemas. Babylon objects are not a public API. This shapes every tooling choice below — fast feedback, clean error messages, and tight package-boundary enforcement matter more than stack flexibility.

## Tooling Decisions

### Formatter: **Biome**

**Why**: Plan §3 default. Single tool for format + lint + import-sort across TS/TSX/JSON/JSONC means one config, one PostToolUse hook, one CI step. Biome's <10ms format pass is fast enough for PostToolUse on every Edit/Write without slowing the agent. Zero-config defaults match the project's YAGNI-first ethos.

**Considered**:
- **oxfmt + oxlint**: faster than Biome (sub-ms), same Oxc toolchain. Tempting for the agentic-fit speed criterion. Rejected because (a) oxlint's rule coverage is still narrower than Biome's for TS-heavy monorepos as of April 2026, (b) running format and lint as one tool reduces config surface and PostToolUse hook count, (c) Biome's CLI exit codes and error messages are better tuned for agent self-correction. Worth re-evaluating at Phase 8 if oxlint's coverage closes the gap.
- **Prettier**: too slow for PostToolUse (~100–300ms). Acceptable for pre-commit only. Loses to Biome on speed without offering anything Biome lacks for this stack.

**Agentic fit**: Excellent. Sub-10ms PostToolUse, zero-config, identical local + CI output, exits 1 cleanly on errors.

### Linter: **Biome**

**Why**: Same toolchain as the formatter. Pairing Biome lint with Biome format means one binary, one config (`biome.json`), one cache. The plan's contracts-first architecture is enforced separately by the `package-boundary-enforcer` agent (`.claude/agents/`), so lint rules can stay general (unused imports, unsafe types, suspicious patterns) without needing custom plugins.

**Considered**:
- **oxlint**: see above — rule-coverage gap.
- **ESLint + @typescript-eslint**: rejected. The plan does not need ESLint's plugin ecosystem (no Next.js, no React 19 server-component plugins, no i18n plugins). ESLint's config maze and slower runtime would hurt agentic iteration.

**Agentic fit**: Excellent. Same speed and message quality as Biome format.

### Test Framework: **Vitest** (unit/integration) + **Playwright** (e2e)

**Why**: Plan §3 default. Vitest is the universal-rules baseline (`testing.md` mandates it). Native ESM, native TS, sub-second feedback for TDD, watch mode for the red→green cycle. Co-located `foo.ts` ↔ `foo.test.ts` matches the universal rule.

Playwright is added as a separate stack only at Phase 3 (smoke test for the 2D renderer's full match loop) and Phase 4 (network smoke). The plan intentionally separates unit/integration tests (Vitest) from end-to-end browser tests (Playwright) — different latency budgets, different failure modes.

**Considered**:
- **Jest**: rejected for new-project default. CJS-first, slow cold start, ESM support requires extra config. Vitest wins on every agentic criterion.
- **node:test**: too minimal for a multi-package monorepo with tRPC + Colyseus + ONNX integration tests.

**Agentic fit**: Excellent. Vitest's error output is precise enough for an agent to diagnose without re-reading the test.

### Git Hooks: **Lefthook**

**Why**: Plan §3 default. Workshop162 standard. Solo developer today, but the project will accept OSS contributions after v1, so a hook runner that survives multiple contributors matters. Lefthook's parallel hook execution keeps pre-commit time low even as we stack `tsc --noEmit`, `biome check`, and the file-size enforcer.

Pre-commit will run:
1. `tsc --noEmit` (typecheck — always, blocks broken TS from reaching the repo)
2. `biome check` (format + lint in one pass)
3. File-size enforcer (mirrors the PostToolUse hook so commit is deterministic with edit-time enforcement)

Tests are NOT in pre-commit — Phase 1's deterministic-sim spike includes long-running benchmarks and the RL training tests will be slow. CI runs the full gate; pre-commit runs fast checks only.

**Considered**:
- **PostToolUse hooks only**: tempting for solo dev, but the plan's universal rules already mandate "no commits with failing tests" — that needs an enforcer at commit time, not just edit time.
- **Husky**: acceptable. Lefthook chosen for parallel execution and Workshop162 convention.
- **simple-git-hooks**: too minimal for a project with this many CI checks.

**Agentic fit**: Good. Pre-commit feedback is delayed relative to PostToolUse, but Lefthook's parallel execution keeps it fast enough to not break flow.

### Build: **Vite (apps/web), tsc (packages), tsup or esbuild (apps/server) — TBD at Phase 0 close**

The plan locks Vite for `apps/web`. Library packages will compile via `tsc -b` for now (zero-config, deterministic). `apps/server` build approach (tsup, esbuild, or plain `tsc`) deferred to Phase 4 when the server actually has runtime code beyond the Phase 0 stub. **No churn — defaults adopted, decision recorded for revisit.**

### Package Manager: **pnpm 10**

Plan-locked. No re-evaluation.

### Python: **uv** + **ruff** + **pytest**

`uv` for Python 3.12 environment + dependency management (plan §8 default; user confirmed). `ruff` for format + lint (Python's Biome equivalent — fast, zero-config, single binary). `pytest` for tests with the marker convention from the plan (`-m bridge`, `-m reward`, `-m eval`, `-m bc`).

## Architecture-Specific Configuration

The plan's contracts-first rule means a generic file-size hook is not enough. The `.claude/agents/package-boundary-enforcer.md` agent (populated, not stub) encodes the exact allowed import graph between `schemas`, `sim-core`, `physics-*`, `render-*`, `policy-runtime`, `bots`, `rl-bridge`, `apps/server`, and `apps/web`. It is read-only and reports violations. Run it after any edit that touches imports.

## Hooks Wired Up

`.claude/settings.json` has two PostToolUse hooks on `Edit|Write`:

1. **File-size enforcer** — extends the universal-rules version with `.tsx` component limits (150 soft / 250 hard) on top of the `.ts` source limits (200 / 300).
2. **Biome formatter** — runs `pnpm -s biome format --write` on the touched file (TS/TSX/JS/JSX/JSON/JSONC/CSS), silent unless it errors. Skips `node_modules`, `dist`, and `.baseline-tmp`.

`lefthook.yml` (configured in the project) adds pre-commit checks: `tsc --noEmit`, `biome check`, file-size enforcer.

## Decisions Deferred

- **`apps/server` build tool**: Phase 4. Default to plain `tsc` until there's a reason to bundle.
- **Linter rule tuning**: ship Biome defaults at Phase 0, tighten in Phase 1 once `sim-core` has real code.
- **Coverage tooling**: Vitest's built-in `--coverage` is fine for v1; revisit if Phase 6 RL eval needs coverage gating.

## What Worked / What Didn't

*(Update after the project has been running a while.)*

## References

- `nimbalyst-local/plans/vex-override-simulator-v2.md` — canonical build plan
- `decisions/seed-getpumped-v3.md` — calibration: similar TypeScript pnpm monorepo with strict architectural boundaries
- `docs/AGENTIC-TOOLING-CRITERIA.md` — evaluation framework

# Decision Records

This directory contains bootstrap decision records — structured notes on what agentic-first tooling was chosen for each project and why.

## Purpose

These records serve two purposes:

1. **Institutional memory** — understand why a project uses a particular tool, not just what it uses
2. **Future bootstrap input** — when bootstrapping a new project with a similar profile, the agent reads matching records to understand what worked and what didn't

## The key rule: consider but never blindly copy

A decision record is not a prescription. It's input.

When bootstrapping a new project, the agent finds records with matching tags, reads what similar projects chose, and then asks: "Is that still the best choice? Has the ecosystem changed? Does this project have different constraints?"

The records show trends. If 5 TypeScript library projects all chose Vitest, that's meaningful signal. But the bootstrap agent still evaluates whether Vitest is right for the new project, rather than copying the choice because "everyone else did it."

## Record format

```markdown
---
project: project-name
date: YYYY-MM-DD
tags: [typescript, monorepo, pnpm, nextjs, ...] ← project characteristics
---

# Project Name — Bootstrap Decisions

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
(Updated after the project has been running for a while)
```

## Tagging conventions

Tags describe the project profile and enable matching. Use lowercase, hyphenated where multi-word.

**Project type**: `typescript`, `javascript`, `library`, `app`, `cli`, `mcp-server`

**Structure**: `monorepo`, `single-package`, `pnpm-workspace`, `turbo`, `nx`

**Framework**: `nextjs`, `vite`, `remix`, `express`, `fastify`, `hono`, `node`

**UI**: `react`, `vue`, `svelte`, `expo`, `react-native`, `no-ui`, `threejs`

**Backend**: `trpc`, `graphql`, `rest`

**Database**: `drizzle`, `prisma`, `sqlite`, `postgres`, `mysql`, `no-db`

**Auth**: `clerk`, `next-auth`, `lucia`, `no-auth`

**Team**: `solo`, `small-team`, `team`

**Other**: `greenfield`, `brownfield`, `saas`, `wasm`, `webgpu`

## Seeded records

The following records were seeded from existing projects before the first automated bootstrap run:

- `seed-getpumped-v3.md` — GetPumped v3 (PumpTrack SaaS, Next.js + Expo monorepo)
- `seed-agenticnc.md` — AgentiCNC (millwork CAD platform, Vite + Three.js monorepo)

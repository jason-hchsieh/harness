# acme-api

HTTP JSON API serving the Acme customer dashboard. Node/TypeScript, pnpm workspace, deployed to Cloudflare Workers.

## Critical Rules

- **NEVER** import from `src/db/` inside `src/handlers/` — go through `src/services/`.
- **IMPORTANT:** run `pnpm gen:types` after editing `schema.prisma`; handlers won't compile otherwise.

## Commands

- Install: `pnpm install` (NOT `npm install` — lockfile is pnpm).
- Dev server: `pnpm dev` (wrangler, port 8787).
- Run single test: `pnpm test <path>` — e.g. `pnpm test src/handlers/users.test.ts`.
- Lint / typecheck: `pnpm lint && pnpm tsc --noEmit`.
- Build: `pnpm build` (bundles to `dist/worker.js`).
- Migrations: `pnpm db:migrate` (local D1), `pnpm db:migrate:prod` (requires approval).

## Project Structure

- `src/handlers/` — HTTP handlers. One file per resource. Template: `handlers/health.ts`.
- `src/services/` — business logic. Handlers call services; services call `src/db/`.
- `src/db/` — Prisma client + query helpers. Do not import from handlers.
- `src/lib/` — cross-cutting utilities (logging, auth, errors).
- `tests/integration/` — run against a local D1 instance. Start with `pnpm db:migrate` first.

## Code Style

- TypeScript strict mode. No `any` — use `unknown` + narrowing.
- ESM only (`"type": "module"`). Import with explicit `.js` extensions in source (TS compiles to JS).
- 2-space indentation, single quotes, trailing commas.
- Naming: `camelCase` for variables/functions, `PascalCase` for types/classes, `kebab-case` for file names and URL paths.

## Testing

- Framework: Vitest. Integration tests use `miniflare` to emulate Workers runtime.
- Prefer fakes over mocks. Mocks are allowed only for third-party HTTP (use `msw`).
- Every handler must have at least one integration test covering 200 + 4xx + 5xx paths.
- Coverage target: 80% line, 70% branch (enforced in CI).

## Git & PR Workflow

- Branch naming: `<type>/<scope>-<summary>` — e.g. `fix/auth-session-timeout`.
- Commit format: Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`).
- Before PR: run `pnpm lint && pnpm test && pnpm tsc --noEmit`. CI runs the same; failing locally means failing in CI.
- Squash-merge only. PR title becomes the commit message.

## Gotchas

- Cloudflare Workers have no `fs`, `net`, or `child_process`. Code that passes locally on Node may fail in Workers runtime — always run `pnpm dev` before PR.
- Request bodies >100MB are rejected by Workers silently. Use streaming uploads via R2 for anything large.
- `Date.now()` in Workers returns the time of the first request in a request lifecycle, not wall time. Use `performance.now()` for timing.
- `pnpm db:migrate:prod` requires the `PRODUCTION_D1_TOKEN` env var set by ops; do not run it from dev machines without coordination.

## References

- @README.md
- @AGENTS.md
- Architecture deep dive: @docs/architecture.md
- On-call runbook: @docs/runbook.md

<!-- maintainer notes: last reviewed 2026-04-15. Target ≤150 lines; currently ~70. -->

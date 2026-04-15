# Anti-patterns (bad vs good)

Short catalog of CLAUDE.md anti-patterns matched by the rubric, paired with their fixes. Use this as a reference when writing `Fix:` lines in audit reports, and as teaching material for `create` mode.

## A1 — Restating the tech stack (R10)

**Bad**
```markdown
## Tech Stack
We use React, TypeScript, Node.js, PostgreSQL, and Redis.
```

**Good** (omit entirely — Claude reads `package.json`)

If there's a project-specific nuance, state only the nuance:
```markdown
## Tech Stack
- Node pinned to 22.x via `.nvmrc` — `nvm use` before running scripts.
- Redis is required locally for integration tests; `docker compose up redis` first.
```

## A2 — File-by-file description (R10)

**Bad**
```markdown
## Files
- `src/index.ts` — entry point
- `src/routes.ts` — route definitions
- `src/handlers/users.ts` — user handlers
- `src/handlers/posts.ts` — post handlers
- `src/db.ts` — database connection
```

**Good** — describe boundaries, not files:
```markdown
## Project Structure
- `src/handlers/` — HTTP handlers. MUST NOT import `src/db/` directly; go through `src/services/`.
- New endpoint? Copy `src/handlers/health.ts` as a template.
```

## A3 — Vague instructions (R08)

**Bad**
- `Write clean code.`
- `Handle errors properly.`
- `Consider performance.`

**Good**
- `Keep functions ≤50 lines; split if cyclomatic complexity > 10.`
- `Wrap async operations in try/catch; log via `logger.error(err, {ctx})`.`
- `Avoid N+1 queries; prefer batched `db.loadMany()`.`

## A4 — Declarative voice (R09)

**Bad**
- `We use 2-space indentation.`
- `Tests should be run before committing.`
- `Branches should be named with a prefix.`

**Good**
- `Use 2-space indentation.`
- `Run `pnpm test` before committing.`
- `Name branches `<type>/<scope>-<summary>`, e.g. `fix/auth-session-timeout`.`

## A5 — Embedded tutorial (R11)

**Bad** (400-line `## API Design Guide` explaining REST conventions)

**Good**
```markdown
## API Design
See @docs/api-conventions.md for endpoint naming, versioning, and pagination rules.
Deviations must be approved in PR review.
```

## A6 — Emphasis inflation (R13)

**Bad**
```markdown
- **IMPORTANT**: Use TypeScript strict mode.
- **IMPORTANT**: Run lint before commit.
- **IMPORTANT**: Name branches with a prefix.
- **YOU MUST**: Write tests for new endpoints.
- **NEVER**: commit `.env` files.
- **ALWAYS**: run migrations after schema change.
- **MUST**: use 2-space indent.
```

**Good** — reserve emphasis for ≤5 truly non-negotiable rules, and promote imperative voice for the rest:
```markdown
## Critical Rules
- **NEVER** commit `.env` files (hook enforces; this rule documents intent).
- **IMPORTANT:** run migrations after schema change before starting dev server.

## Code Style
- Use TypeScript strict mode.
- Use 2-space indentation.
- Run `pnpm lint` before commit.
- Write tests for new endpoints.
```

## A7 — Frontmatter on CLAUDE.md (R20)

**Bad**
```markdown
---
name: my-project
description: A cool project
---

# My Project
...
```

**Good** — CLAUDE.md is pure markdown. Delete the frontmatter block entirely. (Frontmatter is for `SKILL.md` and `AGENTS.md`, not CLAUDE.md.)

## A8 — Committed secrets (R12)

**Bad**
```markdown
## Dev Setup
Set `OPENAI_API_KEY=sk-abc123xyz789...` in your shell.
```

**Good**
```markdown
## Dev Setup
Copy `.env.example` to `.env` and fill in keys (ask in #eng-onboarding).
```

If a secret has been committed, the fix is NOT to move it — it's to rotate the secret AND scrub git history.

## A9 — Double-maintaining with AGENTS.md (R17)

**Bad** — `AGENTS.md` has 80 lines of conventions, `CLAUDE.md` has the same 80 lines plus 10 more. Two sources of truth.

**Good**
```markdown
@AGENTS.md

## Claude Code specific

- Use plan mode for changes under `src/billing/` (regulatory code path).
- When compacting, preserve the list of files touched so the next turn has context.
```

## A10 — Hook-worthy rule left advisory (R15)

**Bad**
```markdown
- **MUST** never commit files containing API keys.
- **ALWAYS** run `pnpm lint` after editing any `.ts` file.
```

**Good** — move deterministic enforcement to hooks / permissions; leave advisory note only:
```markdown
## Automation (enforced elsewhere)
- `.claude/settings.json` runs lint after every edit in `src/**/*.ts`.
- Pre-commit hook blocks commits matching secret patterns.
```

And add the actual hook to `.claude/settings.json` / git pre-commit. CLAUDE.md is advisory; hooks are deterministic.

## A11 — Personal content in shared file (R18)

**Bad** (in `./CLAUDE.md`, checked into git)
```markdown
My sandbox API is at http://localhost:3789.
I prefer verbose output — always pass `--verbose` to test commands.
```

**Good** — move personal content to `./CLAUDE.local.md` (gitignored) or `~/.claude/CLAUDE.md`. The shared project file should speak for the team.

## A12 — Horizontal-rule decoration (R05)

**Bad**
```markdown
## Commands

---

## Project Structure

---

## Testing
```

**Good** — delete the `---` lines. Markdown headings already delimit sections.

## A13 — HTML comments with model-facing instructions (R19)

**Bad**
```markdown
<!-- Remember: always use the new auth middleware for protected routes -->
```

The model will never see this in Claude Code v2.1+; the comment is stripped before injection.

**Good** — put the instruction in regular prose, or if it's maintainer-only, phrase it as a note to humans:
```markdown
<!-- maintainer: this section regenerated from AGENTS.md on 2026-04-15; manual edits will be overwritten -->
```

## A14 — Oversized single section (R04)

**Bad** — `## Testing` section runs 87 lines with framework conventions, mock policies, coverage thresholds, CI interplay, and flaky-test handling.

**Good** — extract to a linked reference:
```markdown
## Testing

See @docs/testing.md for framework, mocks, coverage, and CI.

- Fast path: `pnpm test <path>` runs a single file.
- Integration tests need `docker compose up` first.
```

## A15 — Padding to hit a length target

**Bad** — filler like "We believe in clean code and good engineering practices" added to bulk up a thin file.

**Good** — a 40-line CLAUDE.md that's all signal beats a 150-line CLAUDE.md that's 60% filler. Length is not a quality metric.

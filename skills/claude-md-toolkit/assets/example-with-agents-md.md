@AGENTS.md

# Claude Code specific

This project uses AGENTS.md as the single source of truth for build commands, code style, and architecture. Everything below is Claude Code-specific behavior that doesn't belong in AGENTS.md because other agents don't have these primitives.

## Plan mode

- Use plan mode for any change under `src/billing/` — this code path is in a regulated audit scope.
- Use plan mode for migrations (`db/migrations/`) — destructive changes need explicit review before apply.

## Compaction hints

When `/compact` fires mid-session, preserve:

- The list of files modified in this session (re-inject summaries so the next turn has context).
- Any decisions the user made about architectural trade-offs.

Safe to drop: verbose tool outputs, search results, build logs.

## Subagent delegation

- Searches spanning >5 files: delegate to the `Explore` subagent. Protects the main context window.
- Independent research questions (e.g. "how does feature X compare to alternative Y"): delegate to `general-purpose` subagent.

## Skills

- PDF parsing: use the `pdf` skill.
- Slash commands starting with `/` are user-invocable skills — don't re-implement them inline.

## .claude/rules

Path-scoped rules live in `.claude/rules/`:
- `api.md` (paths: `src/handlers/**`) — handler conventions
- `db.md` (paths: `src/db/**`, `prisma/**`) — schema & query rules
- `billing.md` (paths: `src/billing/**`) — regulated-code rules

These load automatically when Claude touches matching files. Don't duplicate their content here.

<!-- Requires Claude Code v2.1+ for path-scoped rules and HTML comment stripping. -->

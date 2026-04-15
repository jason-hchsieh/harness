# AGENTS.md bridge

How `claude-md-toolkit` handles repos that already have an `AGENTS.md`. Used by `mode-create.md` (skip duplication) and `mode-audit.md` (R17 bridging check).

## Why this matters

`AGENTS.md` is an open standard adopted by OpenAI Codex, Cursor, Copilot, Windsurf, Gemini CLI, and others. Claude Code does NOT natively read `AGENTS.md` — it reads `CLAUDE.md`. Teams using multiple AI coding tools therefore face a choice:

1. **Duplicate**: maintain AGENTS.md and CLAUDE.md in parallel. Rots fast.
2. **Bridge**: CLAUDE.md imports AGENTS.md via `@AGENTS.md` and adds only Claude-specific content on top.
3. **Fork**: CLAUDE.md intentionally diverges. Requires a one-line justification (e.g. AGENTS.md is aspirational and CLAUDE.md is ground truth, or vice versa).

This toolkit prefers **bridge** as the default — it is the single-source-of-truth pattern Anthropic itself recommends.

## Bridge pattern

Top of CLAUDE.md:

```markdown
@AGENTS.md

## Claude Code specific

- <Claude-specific rule 1>
- <Claude-specific rule 2>
```

The `@AGENTS.md` import expands AGENTS.md's content inline when the session loads. Anything below is Claude-specific: plan mode usage, skill invocation patterns, compact-friendly hints, subagent delegation.

## What belongs below the bridge

Content that is genuinely Claude-specific and has no equivalent in other agents:

- Plan mode directives (`Use plan mode for changes under src/billing/`).
- Compaction guidance (`When compacting, preserve the list of files touched`).
- Skill routing hints (`For PDF tasks, invoke the pdf skill via Skill tool`).
- Subagent delegation rules (`Delegate searches spanning >5 files to Explore subagent`).
- Reference to Claude Code-only features: `.claude/rules/`, `.claude/skills/`, `.claude/settings.json` hooks.

## What does NOT belong below the bridge

- General build / test / lint commands — these go in AGENTS.md (all agents need them).
- Repo architecture — AGENTS.md.
- Code style — AGENTS.md.
- Gotchas — AGENTS.md (unless Claude-specific, e.g. context window quirks).

Rule of thumb: if a non-Claude agent would also benefit, it belongs in AGENTS.md.

## Fork pattern (rare)

If bridging doesn't fit — e.g. AGENTS.md is severely out of date and you don't own it — document the divergence at the top:

```markdown
# CLAUDE.md

Note: This repo has AGENTS.md but it is maintained by the vendor team and frequently
out of date. CLAUDE.md is the ground truth for Claude Code sessions until AGENTS.md
is repaired (tracking: JIRA-1234).

## Commands
...
```

Audit R17 will PASS a fork if a one-line justification is present. It will FAIL if AGENTS.md exists, CLAUDE.md does not import it, AND no divergence rationale appears in the first 10 lines.

## Sizing considerations

- If AGENTS.md is >150 lines, bridging it verbatim may blow CLAUDE.md past its token-effective limit.
- In that case, either (a) trim AGENTS.md (ideal — the toolkit can't help with that), or (b) bridge selectively: import only relevant sections using a local `@docs/agents-subset.md` that references AGENTS.md content. Flag this as MEDIUM in audit with a suggestion to consolidate.

## Detection heuristics (used by audit)

For a given CLAUDE.md at path `P`:

1. Sibling check: `Glob` for `AGENTS.md` in the same directory as `P`.
2. If found, `Grep` the CLAUDE.md for `@AGENTS.md` (anywhere in file, including commented imports).
3. If the import is missing, check the first 10 lines for keywords indicating intentional divergence (`AGENTS.md`, `bridge`, `not using`, `diverge`, `fork`).
4. Emit R17 HIGH if neither condition holds.

Cross-directory AGENTS.md (e.g. repo root AGENTS.md with packages/foo/CLAUDE.md) is not flagged at HIGH — emit as MEDIUM advisory, because a package-level CLAUDE.md may intentionally stand alone.

## Import mechanics

- Claude Code expands `@path` imports at session start, relative to the file containing the import.
- Max official depth is 5 hops; this toolkit flags >2 hops as a maintainability concern.
- External imports (paths outside the repo, e.g. `~/.claude/shared.md`) trigger a user approval dialog on first encounter. If the user declines, imports are permanently disabled for that session/project.
- HTML comments around imports still strip them: `<!-- @AGENTS.md -->` will NOT import; the comment is removed and the import is never resolved.

## When to skip the bridge

Bridge is the default, but skip it when:

- AGENTS.md content is harmful to Claude specifically (e.g. it assumes a different tool's primitives). Fork with rationale.
- The repo has AGENTS.md as a legal/contractual artifact (e.g. open-source governance) that the team doesn't treat as operational. Fork with rationale.
- AGENTS.md is a stub (<10 lines) and CLAUDE.md already contains what would be in AGENTS.md. In this case, either (a) move content to AGENTS.md and bridge, or (b) delete the stub AGENTS.md. Audit flags this as MEDIUM advisory.

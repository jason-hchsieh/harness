---
name: claude-md-toolkit
description: Use when creating, auditing, or improving a CLAUDE.md file. Create scaffolds; audit emits R01-R22 findings; improve applies fixes. First arg is the mode. Also bridges to AGENTS.md when present. Do NOT use for SKILL.md — use skill-toolkit.
argument-hint: "create [scope] | audit <path> | improve <path> [--report=<file>]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
disable-model-invocation: false
---

# claude-md-toolkit

A thin router that dispatches to one of three modes for managing CLAUDE.md files. This file stays small on purpose — each mode's real instructions live in `references/mode-*.md` and are loaded only when that mode runs (progressive disclosure).

## Scope & non-goals

**In scope**: creating new CLAUDE.md files (project / user / local / managed scope), auditing existing CLAUDE.md against a 22-rule rubric, applying audit findings as fixes, and bridging with `AGENTS.md` when present.

**Out of scope**: authoring SKILL.md files (use `skill-toolkit`), authoring hooks or settings.json (different primitives), authoring subagents. When the user's CLAUDE.md contains rules that belong in hooks / permissions, the audit WARNS and points at settings.json but does not edit it.

## Tools (why `Bash` is in `allowed-tools`)

The delegated mode files need `Bash` for three narrow purposes — nothing else:

1. `date -u +"%Y-%m-%dT%H:%M:%SZ"` — UTC timestamp at the top of every audit report.
2. `wc -l` — counting lines for R01 length budgets.
3. `basename`/`dirname` — deriving scope from path for R01 and R18 (e.g. is this `./CLAUDE.md` or `~/.claude/CLAUDE.md`).

The router itself does not invoke `Bash`. It is declared at the router level so modes loaded via `Read references/mode-*.md` inherit the permission. No other shell usage is permitted.

## Always load first

Before running any mode, read the shared rubric so the same validation rules apply everywhere:

- `references/rubric.md` — the 22 rules (always load)

## Load on demand

- `references/scope-matrix.md` — load when the target scope matters (any create invocation; any audit applying R01 budgets or R18 scope leakage checks).
- `references/agents-md-bridge.md` — load when a sibling `AGENTS.md` is detected.
- `references/platform-profiles.md` — load when a non-default profile is detected or requested.

## Mode routing

Parse the first whitespace-separated token of `$ARGUMENTS` as the mode. If `$ARGUMENTS` is empty or the mode is unknown, ask the user which mode they want (create / audit / improve) and a target.

### `create [scope]`

Load `references/mode-create.md` and follow it. `scope` defaults to `project` (writes `./CLAUDE.md`). Other valid scopes: `user` (`~/.claude/CLAUDE.md`), `local` (`./CLAUDE.local.md`, ensure `.gitignore`), `managed` (warn about OS-specific paths and MDM deployment). Intent capture asks up to 5 questions, then scaffolds from `assets/skeleton.md` and self-checks against the rubric.

### `audit <path>`

Load `references/mode-audit.md` and follow it. **Read-only** — MUST NOT `Write` or `Edit`. `<path>` can be a CLAUDE.md, CLAUDE.local.md, or a repo root (in which case discover all CLAUDE.md files in the tree). If `AGENTS.md` is found next to the target, also check R17 (bridging).

### `improve <path> [--report=<file>]`

Load `references/mode-improve.md` and follow it. If no report given, run audit inline first. Apply minimal `Edit` diffs keyed to findings (CRITICAL → HIGH → MEDIUM; ask for LOW). Re-audit and report residual.

## Handoff rules between modes

- After `create`: suggest `claude-md-toolkit audit <path>` as an independent sanity check.
- After `audit` with CRITICAL/HIGH findings: suggest `claude-md-toolkit improve <path> --report=<report>`.
- After `improve`: always re-audit and report the residual.

## Anticipated failure modes (cross-mode)

Not yet validated — update as real failures are discovered.

- **Audit mode's read-only contract is prose, not tool-enforced.** `allowed-tools` includes `Write`/`Edit` because create/improve need them. In audit mode, you must self-enforce the constraint.
- **CLAUDE.md has NO frontmatter.** If the target file starts with `---`, that is an R20 HIGH violation — do not mistake it for a SKILL.md.
- **Don't double-maintain with AGENTS.md.** When AGENTS.md exists, the preferred pattern is `@AGENTS.md` import plus Claude-specific addenda — not a second copy.
- **Hard enforcement belongs elsewhere.** Rules like "always run lint" or "never commit secrets" are advisory in CLAUDE.md; point the user at hooks / permissions instead.
- **Scope confusion.** A `./CLAUDE.md` with personal shell aliases is an R18 violation — belongs in `~/.claude/CLAUDE.md` or `./CLAUDE.local.md`.

---
name: skill-toolkit
description: Use when the user wants to CREATE, AUDIT, or IMPROVE a Claude Agent Skill. Create scaffolds a new SKILL.md. Audit runs the R01-R20 rubric and emits severity-tagged findings. Improve applies fixes from an audit report. Mode is the first argument.
argument-hint: "create <name> | audit <path> | improve <path> [--report=<file>]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
disable-model-invocation: false
---

# skill-toolkit

A thin router that dispatches to one of three modes for managing Claude Agent Skills. This file stays small on purpose — each mode's real instructions live in `references/mode-*.md` and are loaded only when that mode runs (progressive disclosure).

## Scope & non-goals

**In scope**: creating new SKILL.md files, auditing existing skills against a 20-rule rubric, applying audit findings as fixes.

**Out of scope**: building plugins end-to-end (this only handles skills), installing/distributing skills (use `/reload-plugins` yourself), managing subagents or slash commands (different primitives).

## Tools (why `Bash` is in `allowed-tools`)

The delegated mode files need `Bash` for two narrow purposes — nothing else:

1. `date -u +"%Y-%m-%dT%H:%M:%SZ"` — UTC timestamp at the top of every audit report (see `references/report-template.md`).
2. `basename`/`dirname` — deriving a skill's directory name for the R01 check ("`name` must equal containing directory name") in `mode-audit.md`.

The router itself does not invoke `Bash`. It is declared at the router level so modes loaded via `Read references/mode-*.md` inherit the permission. No other shell usage is permitted.

## Always load first

Before running any mode, read the shared rubric so the same validation rules apply everywhere:

- `references/rubric.md` — the 20 rules (always load)
- `references/frontmatter-schema.md` — per-field spec (load when needed)

## Mode routing

Parse the first whitespace-separated token of `$ARGUMENTS` as the mode. If `$ARGUMENTS` is empty or the mode is unknown, ask the user which mode they want (create / audit / improve) and a target.

### `create [name]`

Load `references/mode-create.md` and follow it. Default scaffold: `.claude/skills/<name>/`.

### `audit <path>`

Load `references/mode-audit.md` and follow it. **Read-only** — MUST NOT `Write` or `Edit`. `<path>` can be a SKILL.md, a skill directory, or a plugin root.

### `improve <path> [--report=<file>]`

Load `references/mode-improve.md` and follow it. If no report given, run audit inline first.

## Handoff rules between modes

- After `create`: suggest `skill-toolkit audit <path-to-new-skill>` as an independent sanity check.
- After `audit` with CRITICAL/HIGH findings: suggest `skill-toolkit improve <path> --report=<report>`.
- After `improve`: always re-audit and report the residual.

## Anticipated failure modes (cross-mode)

Not yet validated — update as real failures are discovered.

- **Audit mode's read-only contract is prose, not tool-enforced.** `allowed-tools` includes `Write`/`Edit` because create/improve need them. In audit mode, you must self-enforce the constraint.
- **Don't railroad generated skills.** When creating a skill in create mode, use goals+constraints in the body — not prescriptive step-by-step phases. See `references/anti-patterns.md`.
- **Only the 13 documented frontmatter fields exist.** Don't invent new ones.

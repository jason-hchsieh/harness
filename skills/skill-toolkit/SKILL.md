---
name: skill-toolkit
description: Use when the user wants to CREATE, AUDIT, or IMPROVE a Claude Agent Skill. Create scaffolds a new SKILL.md under .claude/skills/ after a short intent interview. Audit runs the R01-R20 rubric and emits a read-only severity-tagged findings report covering all 13 frontmatter fields. Improve applies minimal-diff fixes from an audit report. Pass create|audit|improve as the first argument.
argument-hint: "create <name> | audit <path> | improve <path> [--report=<file>]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
disable-model-invocation: false
---

# skill-toolkit

A thin router that dispatches to one of three modes for managing Claude Agent Skills. This file stays small on purpose — each mode's real instructions live in `references/mode-*.md` and are loaded only when that mode runs (progressive disclosure).

## Scope & non-goals

**In scope**: creating new SKILL.md files, auditing existing skills against a 20-rule rubric, applying audit findings as fixes.

**Out of scope**: building plugins end-to-end (this only handles skills), installing/distributing skills (use `/reload-plugins` yourself), managing subagents or slash commands (different primitives).

## Always load first

Before running any mode, **read the shared rubric** so the same validation rules apply everywhere:

```
Read: references/rubric.md
```

Also keep `references/frontmatter-schema.md` on hand for precise per-field rules — load it when audit or create needs the exact limits.

## Mode routing

Parse the first whitespace-separated token of `$ARGUMENTS` as the mode. If `$ARGUMENTS` is empty or the mode is unknown, ask the user which mode they want (create / audit / improve) and a target.

### `create [name]`

1. Load `references/mode-create.md` and follow its phases.
2. You have `Write` and `Edit` available. Default scaffold location is `.claude/skills/<name>/` in the current working directory.
3. After writing files, remind the user to run `/reload-plugins` and optionally run `skill-toolkit audit` on the new skill as a sanity check.

### `audit <path>`

1. Load `references/mode-audit.md` and follow its phases.
2. **Read-only mode.** You MUST NOT invoke `Write` or `Edit` in this mode. The audit report is the only output. If the user asks for fixes, instruct them to run `skill-toolkit improve <path> --report=<this-report>` instead — do not apply fixes yourself during audit.
3. `<path>` may be a SKILL.md file, a skill directory (containing SKILL.md), or a plugin root (recurse one level into `skills/`).
4. Emit findings using `references/report-template.md`.

### `improve <path> [--report=<file>]`

1. Load `references/mode-improve.md` and follow its phases.
2. If `--report=<file>` is given, read that audit report and apply its findings. If not, run the audit logic inline first, then apply findings.
3. Use `Edit` for minimal diffs. Re-audit after applying fixes and report any residual findings.
4. Remind the user to run `/reload-plugins` when done.

## Handoff rules between modes

- After `create`: suggest `skill-toolkit audit <path-to-new-skill>` as an independent sanity check.
- After `audit` with CRITICAL/HIGH findings: suggest `skill-toolkit improve <path> --report=<report>`.
- After `improve`: always re-audit and report the residual.

## Anti-patterns to avoid (for any mode)

- Do not invent frontmatter fields beyond the 13 documented in `references/frontmatter-schema.md`.
- Do not silently rewrite a user's skill body in `improve` mode — apply minimal diffs keyed to specific findings.
- Do not emit audit findings without a rule id (`R01`..`R20`), evidence, and confidence level.
- Do not claim `disable-model-invocation: true` is needed unless the skill has destructive side effects.

## Why this skill exists (and why it is a skill, not a subagent)

Skills auto-trigger from user intent via description matching; subagents only fire when a parent agent delegates. Because the user will often just say "help me make a skill for X" or "audit this SKILL.md", a skill is the right primitive — and it can still delegate to a subagent via `context: fork` inside a specific phase if the work is heavy and read-only.

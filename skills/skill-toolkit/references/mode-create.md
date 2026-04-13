# Mode: create

Scaffold a new Claude Agent Skill from scratch.

## Goal

Produce a valid, rubric-passing SKILL.md under `.claude/skills/<name>/` that is ready for `/reload-plugins`.

## Input

`$ARGUMENTS` after `create` is an optional skill name (e.g. `create hello-world`). If absent, ask the user.

## Specifications

### Frontmatter output contract

The generated SKILL.md frontmatter must include at minimum:

```yaml
---
name: <name>                    # MUST match directory basename
description: Use when ...       # ≤250 visible chars, front-load trigger words
allowed-tools: Read, Glob, Grep # narrow; add Write/Edit only if skill writes files
disable-model-invocation: false # explicit; true only for destructive side effects
---
```

Add `argument-hint` if the body references `$ARGUMENTS`. Only add advanced fields (`context`, `agent`, `model`, `effort`, `shell`, `hooks`, `paths`) if the user explicitly needs them. See `frontmatter-schema.md` for all 13 fields.

### Body structure

Use `assets/skeleton.md` as the starting template. The generated skill's body should use the goals+constraints pattern:

- **Goal** — what does "done" look like
- **Constraints** — what pushes Claude out of default behavior
- **Gotchas** — failure modes specific to this skill's domain (can be empty initially)

Body must be ≤500 lines. If any section exceeds ~50 lines, move it to `references/` and link.

## Constraints

- Scaffold to `.claude/skills/<name>/` by default. Ask before writing elsewhere.
- If `.claude/skills/<name>/` already exists, STOP — ask to overwrite, rename, or switch to `improve`.
- Only the 13 fields in `frontmatter-schema.md`. Do not invent frontmatter.
- Every path mentioned in the body must resolve (R16). Create referenced files before finishing.

## Workflow

This mode is genuinely sequential — each step depends on the previous:

1. **Capture intent** — up to 5 questions: purpose, trigger phrases, inputs/outputs, tool needs, side effects. Skip what the user already answered.
2. **Check for collisions** — search existing skills for naming conflicts and description overlap (R19).
3. **Scaffold + write** — fill in `skeleton.md` with intent. Write the SKILL.md.
4. **Self-check** — run R01–R20 from `rubric.md`. Fix CRITICAL/HIGH in place.
5. **Handoff** — report the path, suggest audit, remind `/reload-plugins`.

## Anticipated failure modes

These are hypothesized — not yet validated by real runs.

- **Descriptions that are too long.** Count chars before committing. Target ≤250.
- **Forgetting `disable-model-invocation`.** Always set explicitly.
- **Railroading the generated skill.** Use goals+constraints in the output, not step-by-step phases.
- **Overly broad `allowed-tools`.** Start narrow; add Bash only with justification.
- **Dangling references.** Create every file you reference in the body.

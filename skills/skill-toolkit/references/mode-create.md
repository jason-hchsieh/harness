# Mode: create

Scaffold a new Claude Agent Skill from scratch.

## Goal

Produce a valid, rubric-passing SKILL.md under `.claude/skills/<name>/` that is ready for `/reload-plugins`.

## Constraints

- Scaffold to `.claude/skills/<name>/` in the current working directory by default. Ask before writing elsewhere.
- If `.claude/skills/<name>/` already exists, STOP — ask the user to overwrite, rename, or switch to `improve`.
- Frontmatter must use only the 13 fields documented in `frontmatter-schema.md`.
- Body must be ≤500 lines. Move long reference material into `references/` files.
- Every path mentioned in the body must resolve (R16). If you reference a file, create it.
- Description must be ≤250 visible chars, start with "Use when…", and front-load trigger words.
- Use `assets/skeleton.md` as the starting template.

## Workflow

This mode is genuinely sequential — each step depends on the previous:

1. **Capture intent** — ask the user up to 5 questions (purpose, trigger phrases, inputs/outputs, tool needs, side effects). Skip questions the user already answered.
2. **Check for collisions** — search existing skills for naming conflicts and description overlap (R19).
3. **Scaffold + write** — fill in `skeleton.md` with the user's intent. Write the SKILL.md.
4. **Self-check** — run R01–R20 from `rubric.md` against what you just wrote. Fix CRITICAL/HIGH in place.
5. **Handoff** — report the path, suggest audit, remind `/reload-plugins`.

## Gotchas

Claude tends to fail on these during skill creation:

- **Descriptions that are too long.** You'll write a description that perfectly captures the skill — at 400 chars. Count it before committing. Target ≤250.
- **Forgetting `disable-model-invocation`.** Always set it explicitly. Default to `false`. Set `true` only if the skill has destructive side effects (delete, deploy, push).
- **Railroading the generated skill.** The skill you create should follow the same "goals + constraints" pattern, not prescriptive step-by-step phases. Don't reproduce the anti-pattern.
- **Overly broad `allowed-tools`.** Start narrow (Read, Glob, Grep). Add Write/Edit only if the skill writes files. Add Bash only with a justification in the body.
- **Dangling references.** If you mention `references/foo.md` in the body, create the file before finishing. R16 violations are embarrassing.
- **Copying Anthropic's skill-creator verbatim.** Use its structure as a guide, but write the user's skill for their specific need.

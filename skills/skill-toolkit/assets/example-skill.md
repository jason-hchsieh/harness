---
name: hello-world
description: Use when the user asks for a demonstration of a minimal, rubric-passing SKILL.md file. Prints a greeting and explains which rubric rules it satisfies. Do NOT use for real production work — this is a reference example only.
allowed-tools: Read
disable-model-invocation: false
---

# hello-world

A minimal skill that serves as a correct, rubric-passing reference example.

## Goal

Show the user what a small, valid SKILL.md looks like when everything is right.

## Constraints

- Read-only — does not write or edit any files.
- Does not claim to do real work. This skill exists as an example only.

## Rules this skill satisfies

- **R01**: `name: hello-world` matches the containing directory.
- **R02**: description is well under 1024 characters.
- **R03**: description starts with "Use when…".
- **R05**: `disable-model-invocation` is explicitly set.
- **R09**: `allowed-tools` is narrow and read-only — no `Write`, no `Edit`, no `Bash`.
- **R14**: body is tiny, far under the 500-line budget.
- **R17**: no TODOs, no empty sections, no first-person narration, no vague verbs.
- **R18**: description promises a read-only demonstration and the body honors that.

## Anticipated failure modes

None yet — this skill is too simple to have failure modes. This section exists to demonstrate the pattern.

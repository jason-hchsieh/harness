---
name: hello-world
description: Use when the user asks for a demonstration of a minimal, rubric-passing SKILL.md file. Prints a greeting and explains which rubric rules it satisfies. Do NOT use for real production work — this is a reference example only.
allowed-tools: Read
disable-model-invocation: false
---

# hello-world

A minimal skill that serves as a correct, rubric-passing reference example. If you want to know what a small SKILL.md looks like when everything is right, read this.

## Scope & non-goals

**In scope**: demonstrating valid frontmatter, a short body, progressive disclosure (even though this example is too small to need it), and a read-only `allowed-tools` list.

**Out of scope**: doing anything useful. This skill exists as an example only.

## When to use this skill

- When a user types "show me an example of a minimal skill".
- When someone is learning the SKILL.md format and wants a concrete reference.

## How it works

1. Read this file (the user usually just wants to look at it).
2. Print a greeting message summarizing the rules this skill satisfies.

## Rules this skill satisfies

- **R01**: `name: hello-world` matches the containing directory.
- **R02**: description is well under 1024 characters.
- **R03**: description starts with "Use when…".
- **R05**: `disable-model-invocation` is explicitly set.
- **R09**: `allowed-tools` is narrow and read-only — no `Write`, no `Edit`, no `Bash`.
- **R14**: body is tiny, far under the 500-line budget.
- **R17**: no TODOs, no empty sections, no first-person narration, no vague verbs.
- **R18**: description promises a read-only demonstration and the body honors that.

## Anti-patterns this skill avoids

- Does not claim to do real work.
- Does not request `Write` or `Bash` access.
- Does not set `disable-model-invocation: true` because there are no destructive side effects.

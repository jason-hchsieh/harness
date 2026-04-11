# Mode: create

Scaffold a new Claude Agent Skill from scratch. Walk the user through intent capture, write a valid SKILL.md, and self-check it against the rubric before handing off.

**Before running this mode**, make sure you have already loaded `rubric.md` and `frontmatter-schema.md` (the SKILL.md router does this).

## Input contract

- `$ARGUMENTS` after the mode token is optional. If a name is provided (e.g. `create hello-world`), use it as a starting point. Otherwise ask the user.
- Default scaffold location: `.claude/skills/<name>/` in the current working directory.
- If `.claude/skills/<name>/` already exists, STOP and ask the user whether to overwrite, pick a new name, or switch to `improve` mode.

## Phase 1 — Capture intent (≤5 questions)

Ask the user these questions, concisely, in order. Skip a question if the user already answered it in the initial prompt. Do not ask more than 5 questions total — you are gathering enough signal, not writing a requirements doc.

1. **Purpose**: What should this skill do? One sentence.
2. **Trigger**: What would the user say out loud when they want this skill to run? Give one or two example phrasings.
3. **Inputs/outputs**: What does the skill consume (file path, free text, nothing) and what does it produce (files written, a report, an answer)?
4. **Tools**: Which tools will it need? (Read, Write, Edit, Glob, Grep, Bash, WebFetch…) If `Bash`, what commands?
5. **Side effects**: Does it write files, delete things, push to remote, call external APIs? (If yes → `disable-model-invocation: true`.)

Record the answers as a short intent block you will reference in later phases.

## Phase 2 — Research prior art

Use `Glob` and `Grep` to look for collisions or patterns to reuse:

- `Glob` `.claude/skills/*/SKILL.md` and `~/.claude/skills/*/SKILL.md` to list existing skills.
- `Grep` the word the user chose for `name` across those SKILL.md files to detect naming collisions.
- If collisions exist, ask the user to pick a different name.
- Optionally `Grep` descriptions for overlap with the user's proposed trigger phrases (to avoid attention competition per R19).

## Phase 3 — Scaffold the directory

1. Create the directory: `.claude/skills/<name>/`
2. Copy `assets/skeleton.md` into `.claude/skills/<name>/SKILL.md` using `Read` + `Write` (do not shell out).
3. Fill in the frontmatter from the Phase 1 intent block:
   - `name: <name>` (MUST match directory)
   - `description`: write a "Use when…" clause that front-loads the trigger phrases from Q2. Keep it ≤250 visible chars if possible, ≤1024 hard.
   - `allowed-tools`: the narrow list from Q4. If `Bash`, add a one-line justification comment in the body.
   - `disable-model-invocation`: `true` if Q5 indicated side effects, else `false` (always set explicitly).
   - `argument-hint`: if the body will reference `$ARGUMENTS`.
   - Any advanced fields (`context`, `agent`, `paths`, `hooks`) only if the user explicitly needs them.

## Phase 4 — Write the body

Draft a body that uses progressive disclosure. Target ≤300 lines; hard cap 500.

Recommended skeleton (from `assets/skeleton.md`):

1. **One-line purpose** (mirrors description).
2. **Scope & non-goals** (what it does NOT do).
3. **When to use this skill** (concrete phrasings).
4. **How it works** — the actual instructions. Keep each step testable.
5. **Inputs/outputs contract**.
6. **Anti-patterns** (what not to do).
7. **References** — links to files under `references/` for long reference material.

If any single section grows past ~50 lines, move it to a file under `skills/<name>/references/` and link to it from the body. Do not inline long lists, long tables, or long examples.

## Phase 5 — Self-check against the rubric

Run the R01–R20 checks from `rubric.md` against the SKILL.md you just wrote. Fix anything CRITICAL or HIGH in place before finishing. Specifically:

- **R01**: name matches directory name.
- **R02**: description ≤1024 chars (count them).
- **R03**: description contains "Use when…".
- **R09**: allowed-tools is narrow; Bash justified.
- **R14**: body ≤500 lines.
- **R16**: every `references/*` / `scripts/*` / `assets/*` path you mentioned actually exists (create stubs if you referenced a file you haven't written yet).
- **R18**: the description's promise matches what the body actually does.

If you discover a gap (e.g. you referenced `references/foo.md` but haven't written it), create the stub file now — do not leave dangling references.

## Phase 6 — Handoff

Report to the user:

1. The path that was written: `.claude/skills/<name>/SKILL.md`
2. A one-line summary of what the skill does.
3. A reminder to run `/reload-plugins` so Claude Code picks up the new skill.
4. A suggestion to run `skill-toolkit audit .claude/skills/<name>/SKILL.md` as an independent sanity check.

## Things you must not do in create mode

- Do not scaffold a skill outside `.claude/skills/<name>/` without asking the user first.
- Do not invent frontmatter fields beyond the 13 in `frontmatter-schema.md`.
- Do not write a body longer than 500 lines. Progressive disclosure is not optional.
- Do not copy and paste Anthropic's `skill-creator` verbatim — use its structure as a guide but write the user's skill for their specific need.

# SKILL.md frontmatter schema — 13 official fields

Per-field reference for the 13 frontmatter fields recognized by Claude Code, sourced from [shanraisshan/claude-code-best-practice/claude-skills.md](https://raw.githubusercontent.com/shanraisshan/claude-code-best-practice/main/best-practice/claude-skills.md) (v2.1.97, April 2026). Use this as the authoritative per-field spec when writing or auditing a SKILL.md.

All fields live between `---` markers at the top of the file and use YAML syntax.

---

## Core identity

### `name` (required)

- **Purpose**: display name and slash-command identifier.
- **Format**: `^[a-z0-9][a-z0-9-]{0,63}$` — lowercase alphanumerics and hyphens, ≤64 chars.
- **Default**: directory name. If you omit `name`, it is inferred from the containing directory.
- **Rule**: MUST equal the directory name when both are present.
- **Example**: `name: skill-toolkit`

### `description` (required)

- **Purpose**: explains what the skill does and when to use it. Shown in `/` autocomplete. Used by Claude for auto-discovery (description matching against user intent).
- **Length**: hard ≤1024 chars; soft ≤250 visible chars (Claude Code UI truncates the display around there).
- **Structure** (SHOULD):
  - Lead with "Use when…" (or equivalent).
  - Enumerate all valid intents if the skill has multiple modes.
  - End with a "Do NOT use…" disambiguator if sibling skills exist.
  - Front-load concrete nouns and action verbs users would actually say.
- **Example**:
  ```yaml
  description: Use when the user wants to CREATE a new Claude Agent Skill, AUDIT an existing skill against a quality rubric, or IMPROVE a skill based on audit findings.
  ```

---

## Invocation control

### `argument-hint`

- **Purpose**: autocomplete guidance for `$ARGUMENTS`. Displayed after the slash-command name in the UI.
- **Format**: free-form string; typical convention uses angle brackets or pipe-separated alternatives.
- **When to use**: if the skill body references `$ARGUMENTS`.
- **Example**: `argument-hint: "<issue-number>"` or `argument-hint: "create <name> | audit <path>"`

### `disable-model-invocation`

- **Purpose**: when `true`, Claude will NOT automatically invoke the skill — only manual `/skill-name` works.
- **Type**: boolean.
- **Default**: `false`.
- **When to set `true`**: destructive side effects (delete, deploy, push, mutate external state), or any operation you want the user to deliberately ask for.
- **Rule**: if set to `true`, include a one-line rationale in the body.
- **Example**: `disable-model-invocation: true  # destructive: writes to deploy target`

### `user-invocable`

- **Purpose**: when `false`, hides the skill from the `/` menu. The skill becomes background knowledge Claude can still auto-load via description matching.
- **Type**: boolean.
- **Default**: `true`.
- **When to set `false`**: reference-material skills that should load automatically but aren't something the user types.
- **Example**: `user-invocable: false`

---

## Execution parameters

### `model`

- **Purpose**: pin the Claude variant used when the skill runs.
- **Allowed values**: `haiku`, `sonnet`, `opus`.
- **Default**: inherits session default.
- **When to use**: rarely — usually a smell. The session default is almost always correct.
- **Example**: `model: sonnet`

### `effort`

- **Purpose**: override the effort level (how much thinking the model does) for the skill.
- **Allowed values**: `low`, `medium`, `high`, `max`.
- **Default**: inherits session default.
- **When to use**: rarely. Pin only when you have measured it matters.
- **Example**: `effort: high`

### `allowed-tools`

- **Purpose**: tools the skill can invoke without prompting the user for per-use permission.
- **Format**: comma-separated list or YAML list of tool names.
- **Rule**: narrow list preferred. `Bash` is broad and requires a justification in the body. Wildcards are forbidden. Typos fail validation.
- **Example**: `allowed-tools: Read, Write, Edit, Glob, Grep, Bash`

### `shell`

- **Purpose**: command environment for Bash tool invocations.
- **Allowed values**: `bash` (default), `powershell`.
- **Default**: `bash`.
- **When to use**: cross-platform skills targeting Windows PowerShell.
- **Example**: `shell: powershell`

---

## Advanced context

### `context`

- **Purpose**: when set to `fork`, the skill runs inside an isolated subagent (separate context window).
- **Allowed values**: `fork` (currently the only documented value).
- **Default**: inline in main session.
- **When to use**: heavy read-only work where you want to protect the main context from pollution (e.g. a deep audit across many files).
- **Rule**: if `context: fork`, you MUST also set `agent`.
- **Example**: `context: fork`

### `agent`

- **Purpose**: which subagent type to use when `context: fork` is set.
- **Format**: a subagent type name (e.g. `general-purpose`, `Explore`, or a custom agent name).
- **Default**: `general-purpose`.
- **Rule**: has no effect unless `context: fork` is also set. Flag as HIGH if `agent` is set alone.
- **Example**: `agent: Explore`

### `hooks`

- **Purpose**: lifecycle event handlers for the skill (e.g. pre-invoke, post-invoke, on-error).
- **Format**: YAML mapping of event name → executable path.
- **Rule**: each referenced path MUST exist on disk; unknown event names are flagged.
- **Example**:
  ```yaml
  hooks:
    post-invoke: scripts/log-invocation.sh
  ```

### `paths`

- **Purpose**: glob patterns that trigger auto-activation when the user opens or edits a matching file.
- **Format**: YAML list of glob strings.
- **Rule**: must be valid glob syntax. Overly broad globs (`**/*`, `*`) are flagged — they will activate the skill on nearly every file and compete for attention.
- **Example**:
  ```yaml
  paths:
    - "**/SKILL.md"
    - "skills/**/*.md"
  ```

---

## Field summary

| Field                      | Required | Type    | Default            | Common usage          |
|----------------------------|----------|---------|--------------------|-----------------------|
| `name`                     | yes      | string  | directory name     | always                |
| `description`              | yes      | string  | —                  | always                |
| `argument-hint`            | no       | string  | —                  | skills with args      |
| `disable-model-invocation` | no       | boolean | `false`            | destructive skills    |
| `user-invocable`           | no       | boolean | `true`             | background skills     |
| `model`                    | no       | enum    | session default    | rare                  |
| `effort`                   | no       | enum    | session default    | rare                  |
| `allowed-tools`            | no       | list    | all tools prompted | always recommended    |
| `shell`                    | no       | enum    | `bash`             | Windows skills        |
| `context`                  | no       | enum    | inline             | heavy read-only work  |
| `agent`                    | no       | string  | `general-purpose`  | with `context: fork`  |
| `hooks`                    | no       | map     | —                  | lifecycle automation  |
| `paths`                    | no       | list    | —                  | file-triggered skills |

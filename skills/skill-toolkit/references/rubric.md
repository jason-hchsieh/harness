# skill-toolkit rubric (R01–R20)

The single source of truth for validation rules used by `create`, `audit`, and `improve` modes. Every audit finding MUST cite a rule id from this file plus an evidence snippet plus a confidence level.

Rules R01–R13 cover the 13 official frontmatter fields enumerated in [shanraisshan/claude-code-best-practice/claude-skills.md](https://raw.githubusercontent.com/shanraisshan/claude-code-best-practice/main/best-practice/claude-skills.md). Rules R14–R20 cover body-level concerns.

For the precise per-field spec (allowed values, defaults, examples) see `frontmatter-schema.md`.

---

## Frontmatter rules (13 official fields)

### R01 — `name` required + format

- Required field.
- Must match regex `^[a-z0-9][a-z0-9-]{0,63}$` (lowercase alphanumerics and hyphens, ≤64 chars, starts with alphanumeric).
- Must equal the containing directory name (e.g. `skills/my-skill/SKILL.md` → `name: my-skill`).
- **Confidence**: HIGH.
- **Severity if violated**: CRITICAL (skill will not load or will load under the wrong identifier).

### R02 — `description` required + length

- Required field.
- Hard limit: ≤1024 characters.
- Soft limit: ≤250 visible characters (the Claude Code UI truncates the displayed description around there).
- **Confidence**: HIGH.
- **Severity if violated**: CRITICAL for hard-limit violation; HIGH for soft-limit violation.

### R03 — `description` structure

- MUST contain a "Use when…" clause (or equivalent like "Use this when…").
- If sibling skills exist in the same plugin, SHOULD contain a "Do NOT use…" disambiguator.
- SHOULD front-load the key trigger words a user would actually say (concrete nouns, action verbs).
- **Confidence**: MEDIUM.
- **Severity if violated**: HIGH (auto-trigger will be unreliable).

### R04 — `argument-hint`

- If the skill body references `$ARGUMENTS`, this field SHOULD be present for autocomplete.
- Form: a brief human-readable hint, typically in angle brackets or square brackets (e.g. `"<issue-number>"` or `"create <name> | audit <path>"`).
- **Confidence**: MEDIUM.
- **Severity if violated**: LOW (WARN only).

### R05 — `disable-model-invocation`

- SHOULD be explicitly set to `true` or `false` (do not omit).
- If `true`, the skill body MUST contain a one-line rationale explaining why auto-invocation is suppressed (typical reason: destructive side effects).
- **Confidence**: HIGH for the schema check, MEDIUM for the "rationale present" check.
- **Severity if violated**: MEDIUM.

### R06 — `user-invocable`

- Optional. If `false`, the skill is hidden from the `/` menu and becomes background knowledge only.
- If set to `false`, the body prose MUST read as reference material, not as an action the user asks for.
- **Confidence**: MEDIUM.
- **Severity if violated**: MEDIUM.

### R07 — `model`

- Optional. If set, must be one of `haiku`, `sonnet`, `opus`.
- Flag as WARN if set without a one-line justification in the body — the session default is usually correct.
- **Confidence**: HIGH for the enum check.
- **Severity if violated**: LOW (WARN).

### R08 — `effort`

- Optional. If set, must be one of `low`, `medium`, `high`, `max`.
- Flag as WARN if pinned without justification.
- **Confidence**: HIGH for the enum check.
- **Severity if violated**: LOW (WARN).

### R09 — `allowed-tools` minimality

- SHOULD be present and narrow.
- `Bash` requires a one-line justification in the body (it is the broadest tool).
- Wildcards (`*`, `**`) are forbidden.
- Tool names MUST exist in the real tool catalog (typos like `Reads` or `Eidt` are flagged).
- **Confidence**: HIGH for schema/typo check, MEDIUM for "Bash justification present".
- **Severity if violated**: HIGH.

### R10 — `shell`

- Optional. If set, must be `bash` or `powershell`.
- Flag as WARN if the skill is cross-platform but pins a specific shell.
- **Confidence**: HIGH.
- **Severity if violated**: LOW (WARN).

### R11 — `context` + `agent` pair

- Optional. If `context: fork`, then `agent` MUST be specified and MUST name a real subagent type (e.g. `general-purpose`, `Explore`).
- If `agent` is set without `context: fork`, flag as HIGH (the `agent` field has no effect unless `context: fork`).
- **Confidence**: HIGH.
- **Severity if violated**: HIGH.

### R12 — `hooks`

- Optional. If present, each hook entry MUST reference an executable path that exists on disk.
- Unknown hook event names are flagged.
- **Confidence**: HIGH for existence check, MEDIUM for "event name known".
- **Severity if violated**: HIGH.

### R13 — `paths`

- Optional. If present, each entry MUST be valid glob syntax.
- Flag as HIGH if overly broad (e.g. `**/*`, `*`) because this enables auto-activation on nearly every file.
- **Confidence**: HIGH for syntax, MEDIUM for "overly broad".
- **Severity if violated**: HIGH.

---

## Body-level rules

### R14 — Body length budget

Line counts refer to the SKILL.md body (everything after the closing `---` of the frontmatter).

- ≤500 lines: pass.
- 500–600 lines: WARN (MEDIUM).
- 600–850 lines: HIGH.
- >850 lines: CRITICAL.

~850 is Anthropic's own `skill-creator` size — at the ceiling. Anything beyond that is a sign you need progressive disclosure (move content into `references/`).

- **Confidence**: HIGH.

### R15 — Progressive disclosure

- Any body >400 lines MUST link into `references/` files rather than inline long reference material.
- Flag if body >400 lines and zero reference-file links are present.
- **Confidence**: MEDIUM.
- **Severity if violated**: HIGH.

### R16 — Supporting file integrity

- Every path the body references (e.g. `references/foo.md`, `scripts/bar.py`, `assets/baz.png`) MUST resolve to a real file.
- **Confidence**: HIGH.
- **Severity if violated**: HIGH.

### R17 — Anti-patterns

Flag any of the following in the body:

- Literal `TODO`, `FIXME`, `XXX` markers.
- Empty `##` sections (a heading followed immediately by another heading or by only whitespace).
- Vague verbs in headings or instruction text: `handle`, `manage`, `process`, `deal with`, `take care of`.
- First-person narration: `I will`, `I'll`, `my plan is to`.
- **Confidence**: MEDIUM (heuristic regex matching).
- **Severity if violated**: MEDIUM.

See `anti-patterns.md` for bad-vs-good examples.

### R18 — Principle of least surprise

- Stated purpose in `description` MUST match actual behavior described in the body.
- Read-only skills MUST NOT list `Write` or `Edit` in `allowed-tools`.
- Skills with destructive side effects (delete, deploy, push, mutate external state) MUST have `disable-model-invocation: true`.
- **Confidence**: LOW (judgement call, emit as advisory).
- **Severity if violated**: HIGH when the mismatch is unambiguous; LOW when it is an advisory nudge.

### R19 — Trigger non-competition

- Within one plugin, no two skills may share the same top trigger phrase in their descriptions.
- Effectively a no-op when the plugin contains a single skill, but still checked so it catches regressions when a second skill is added.
- **Confidence**: MEDIUM.
- **Severity if violated**: HIGH.

### R20 — Description triggerability

- Description SHOULD contain at least one concrete noun that a user would actually say out loud when asking for help — not purely abstract verbs or filler words.
- Bad example: "Handle various workflows and automate processes."
- Good example: "Create a new SKILL.md file with valid frontmatter."
- **Confidence**: LOW (heuristic).
- **Severity if violated**: LOW (WARN).

---

## Confidence summary

| Confidence | Rules                                                                 |
|------------|-----------------------------------------------------------------------|
| HIGH       | R01, R02, R04 (schema), R05 (schema), R07, R08, R09 (schema), R10, R11, R12 (existence), R13 (syntax), R14, R16 |
| MEDIUM     | R03, R04 (presence heuristic), R05 (rationale), R06, R09 (Bash justification), R12 (event known), R13 (overly broad), R15, R17, R19 |
| LOW        | R18, R20                                                               |

## Severity summary

| Severity | Typical triggers                                                     |
|----------|----------------------------------------------------------------------|
| CRITICAL | R01 violation, R02 hard-limit violation, R14 >850 lines              |
| HIGH     | R02 soft-limit, R03, R09, R11, R12, R13, R14 600–850, R15, R16, R19, R18 unambiguous |
| MEDIUM   | R04, R05, R06, R14 500–600, R17                                      |
| LOW      | R07, R08, R10, R18 advisory, R20                                     |

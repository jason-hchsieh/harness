# Mode: audit

Read-only rubric check against an existing SKILL.md. Emits a severity-tagged findings report.

## Goal

Produce a findings report for one or more SKILL.md files by running every applicable rule from `rubric.md`. The report follows `references/report-template.md`.

## Input

`$ARGUMENTS` after `audit` is a path — one of:
- A `SKILL.md` file (e.g. `.claude/skills/hello/SKILL.md`)
- A skill directory containing `SKILL.md` (e.g. `.claude/skills/hello`)
- A plugin root with `skills/` — audit each SKILL.md, one report section per skill plus cross-skill summary

If path doesn't exist, ask the user. Don't proceed with a partial audit.

## Specifications

### Finding record format

Every finding MUST contain these 6 fields — no exceptions, no shortcuts:

```
### [SEVERITY] Rnn short-rule-name

- **File**: <path>:<line>
- **Evidence**: <verbatim snippet or measurement, e.g. "description is 1187 chars">
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: <one-line actionable suggestion>
```

- `severity`: one of CRITICAL, HIGH, MEDIUM, LOW — per `rubric.md`.
- `confidence`: per `rubric.md`'s per-rule definition (HIGH = deterministic check, LOW = judgement call).
- `location`: always `file:line` format. If the violation spans multiple lines, point to the first offending line.
- `evidence`: a short verbatim snippet from the file or a measurement. Not a paraphrase. E.g. `"name: Hello-World"` (wrong case) or `"body is 623 lines"`.

### Report structure

```
# Skill Audit Report — <skill-name>

Target: <path>
Date: <UTC ISO-8601>
Rubric: harness-toolkit v0.1.0

## Summary
- CRITICAL: <n> | HIGH: <n> | MEDIUM: <n> | LOW: <n>
- Body length: <n> lines (budget 500)
- Description length: <n> chars (hard 1024 / soft 250)
- Frontmatter fields present: <list>
- Frontmatter fields missing (recommended): <list>

## Findings
[findings grouped by severity: CRITICAL → HIGH → MEDIUM → LOW]
[within each group, sorted by rule id ascending]

## Next step
To apply fixes: `skill-toolkit improve <path> --report=<this-report>`
```

See `references/report-template.md` for multi-skill reports and the empty-findings case.

### Empty-findings case

Still emit the full report header with all zeroes. An empty pass report is a valid audit artifact. Do not skip the report just because nothing failed.

## Constraints

- **Read-only.** MUST NOT invoke `Write` or `Edit`. Output to conversation only. If user asks for a file, confirm first (breaks read-only contract).
- **One finding per violation.** Do not collapse multiple violations of the same rule. `improve` mode addresses each individually.
- **No new rule ids.** If a check isn't covered by R01–R20, emit it as advisory under R18 or propose adding the rule to `rubric.md` separately.
- **Don't mix audit and improve.** If the user says "fix this" mid-audit, redirect to `skill-toolkit improve <path>`.

## Anticipated failure modes

These are hypothesized failure patterns — not yet validated by real runs. Update this section as you discover actual failures.

- **Skipping R16 (supporting file integrity)**: you must actually `Glob` every referenced path, not just assume it exists.
- **Miscounting description length**: multi-line YAML descriptions fold whitespace. Count the final resolved string, not the raw YAML lines.
- **False-positive R17 on code blocks**: don't flag `TODO` or `FIXME` inside fenced code blocks — only flag them in prose.
- **R01 directory derivation**: the directory to match is the immediate parent of the SKILL.md file, not the plugin root. `skills/foo/SKILL.md` → name must be `foo`.
- **R18 confidence**: R18 is LOW confidence by definition. Emit as advisory, never as HIGH.

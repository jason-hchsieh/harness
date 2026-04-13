# Mode: audit

Read-only rubric check against an existing SKILL.md. Emits a severity-tagged findings report.

## Goal

Produce a findings report for one or more SKILL.md files by running every applicable rule from `rubric.md`. The report follows `references/report-template.md`.

## Constraints

- **Read-only.** MUST NOT invoke `Write` or `Edit`. The findings report is conversation output only.
- Every finding must include: `rule` (R01–R20), `severity`, `confidence`, `location` (file:line), `evidence`, `fix` suggestion.
- One finding per violation — do not collapse multiple violations of the same rule.
- If the user asks "fix this" mid-audit, redirect to `skill-toolkit improve <path>`. Do not mix audit and improve in one turn.
- Output to conversation by default. If the user explicitly asks for a file, confirm before writing (you'd be breaking read-only).
- Use `references/report-template.md` for the output structure.

## Input

`$ARGUMENTS` after `audit` is a path — one of:
- A `SKILL.md` file
- A skill directory containing `SKILL.md`
- A plugin root with `skills/` — audit each SKILL.md, one report section per skill plus cross-skill summary

If path doesn't exist, ask the user. Don't proceed with a partial audit.

## What to check

Run **R01–R20 from `rubric.md`**. The rubric defines what each rule checks, its confidence level, and its severity. You do not need additional instructions beyond what's in the rubric — apply each rule to the target and emit a Finding when violated.

**Finding record format:**

```
### [SEVERITY] Rnn short-rule-name

- **File**: path:line
- **Evidence**: verbatim snippet or measurement
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: one-line suggestion
```

Group findings by severity (CRITICAL → HIGH → MEDIUM → LOW), then by rule id within each group.

## Report header

Include at the top of every report:
- Target path
- UTC timestamp
- Rubric version (`harness-toolkit v0.1.0`)
- Summary counters: CRITICAL n | HIGH n | MEDIUM n | LOW n
- Key scalars: body line count, description char count

For multi-skill audits (plugin root), add a cross-skill summary table per `report-template.md`.

## Handoff

End with: "To apply fixes, run `skill-toolkit improve <path> --report=<this-report>`"

If zero CRITICAL/HIGH findings: congratulate and suggest `/reload-plugins`.

## Gotchas

Claude tends to fail on these — pay extra attention:

- **Skipping R16 (supporting file integrity)**: you must actually `Glob` every referenced path, not just assume it exists. This is the most commonly missed check.
- **Miscounting description length**: multi-line YAML descriptions fold whitespace. Count the final resolved string, not the raw YAML lines.
- **False-positive R17 on code blocks**: don't flag `TODO` or `FIXME` inside fenced code blocks (` ``` `) — only flag them in prose.
- **R01 directory derivation**: the directory to match is the immediate parent of the SKILL.md file, not the plugin root. `skills/foo/SKILL.md` → name must be `foo`.
- **R18 is a judgement call**: don't flag it at HIGH confidence. It's LOW confidence by definition — emit as advisory.
- **Empty findings are valid**: if nothing fails, still emit the full report header with zeroes. An empty pass report is an audit artifact.

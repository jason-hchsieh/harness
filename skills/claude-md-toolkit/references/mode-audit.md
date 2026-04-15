# Mode: audit

Read-only rubric check against an existing CLAUDE.md. Emits a severity-tagged findings report.

## Goal

Produce a findings report by running every applicable rule from `rubric.md` against the target file. The report follows `references/report-template.md`.

## Input

`$ARGUMENTS` after `audit` is a path — one of:
- A `CLAUDE.md` or `CLAUDE.local.md` file
- A directory containing a CLAUDE.md at its root
- A repo root to sweep (in which case `Glob` for `**/CLAUDE.md` and `**/CLAUDE.local.md`, respecting `.gitignore`, and emit one report section per file plus a sweep summary)

If the path doesn't exist, ask the user — don't partial-audit.

## Specifications

### Scope detection (for R01 and R18)

Derive the scope from the absolute path:
- `~/.claude/CLAUDE.md` → `user`
- `./CLAUDE.local.md` → `local`
- `/etc/claude-code/CLAUDE.md`, macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`, Windows `C:\Program Files\ClaudeCode\CLAUDE.md` → `managed`
- Everything else → `project`

Scope determines which column of R01's length table applies.

### Profile detection (for platform-profiles.md overrides)

Scan the target's directory for signals:
- `package.json` with `workspaces` field, `pnpm-workspace.yaml`, `turbo.json`, or `lerna.json` → `monorepo`
- `pyproject.toml` mentioning `torch`, `tensorflow`, `transformers`, `wandb`, `mlflow`, or a `models/` / `datasets/` / `experiments/` dir at repo root → `ml`
- Presence of `CONTRIBUTING.md` + `LICENSE` + public-looking `README.md` (badges, install section) → `open-source`
- Presence of `SECURITY.md` with compliance keywords (HIPAA / SOC2 / PCI / GDPR) or a `compliance/` dir → `compliance`
- Otherwise → `default`

Only the first matching signal is applied. Emit the detected profile in the report header.

### Finding record format

Every finding MUST contain these 6 fields — no exceptions:

```
### [SEVERITY] Rnn short-rule-name

- **File**: <path>:<line>
- **Evidence**: <verbatim snippet or measurement>
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: <one-line actionable suggestion>
```

- `severity`: one of CRITICAL, HIGH, MEDIUM, LOW — per `rubric.md`.
- `confidence`: per `rubric.md`'s per-rule definition.
- `location`: `file:line` format. If the violation spans multiple lines, point to the first offending line. For file-level measurements (e.g. R01 line count, R13 emphasis total), use `file:1`.
- `evidence`: verbatim snippet or a measurement. Not a paraphrase. E.g. `"IMPORTANT: used 9 times"` or `"line 87: 'Write clean code'"`.
- `fix`: one line, actionable. Not "consider refactoring" — say exactly what to change.

### Report structure

See `references/report-template.md` for the full scaffold. Summary block for each file:

```
# CLAUDE.md Audit Report

Target: <path>
Date: <UTC ISO-8601>
Scope: <project|user|local|managed>
Profile: <default|monorepo|ml|open-source|compliance>
Rubric: claude-md-toolkit v0.1.0

## Summary
- CRITICAL: <n> | HIGH: <n> | MEDIUM: <n> | LOW: <n>
- Line count: <n> (budget for <scope>: PASS ≤<x>, WARN ≤<y>, HIGH ≤<z>, CRITICAL >z)
- Heading count: <h1> h1, <h2> h2, <h3> h3, <h4+> h4+
- Imports: <n> (depth max <d>)
- Emphasis tokens (IMPORTANT/MUST/NEVER/ALWAYS/YOU MUST): <n>
- AGENTS.md sibling: <present | absent>
- `.gitignore` covers CLAUDE.local.md: <yes | no | n/a>

## Findings
[findings grouped by severity: CRITICAL → HIGH → MEDIUM → LOW]
[within each group, sorted by rule id ascending]

## Next step
To apply fixes: `claude-md-toolkit improve <path> --report=<this-report>`
```

### Multi-file sweep

When the input is a repo root, prepend a sweep summary:

```
# CLAUDE.md Sweep Summary — <repo-name>

Target: <repo-root>
Files discovered: <n> (CLAUDE.md: <n>, CLAUDE.local.md: <n>)
Total findings: CRITICAL <n> | HIGH <n> | MEDIUM <n> | LOW <n>

## Per-file summary

| Path | Scope | CRIT | HIGH | MED | LOW | Lines |
|------|-------|------|------|-----|-----|-------|
| ./CLAUDE.md | project | 0 | 2 | 3 | 1 | 178 |
| packages/api/CLAUDE.md | project | 0 | 0 | 1 | 0 | 94 |
```

Then per-file reports follow.

### Empty-findings case

Still emit the full report header with all zeroes. An empty pass report is a valid artifact.

## Constraints

- **Read-only.** MUST NOT invoke `Write` or `Edit`. Output to conversation only. If the user asks for the report as a file mid-audit, acknowledge the request but defer until audit completes, then offer `improve` or a separate explicit Write in a fresh turn.
- **One finding per violation.** Do not collapse multiple R08 vague-phrase hits into a single finding — each instance gets its own record so `improve` can address them individually. Exception: R01 (file-level length) emits one finding per file.
- **No new rule ids.** If a check isn't covered by R01–R22, emit as advisory under R07 (deletability) or propose adding a rule — don't invent R23.
- **Don't mix audit and improve.** If the user says "fix this" mid-audit, finish the audit report first, then redirect to `claude-md-toolkit improve`.
- **Don't audit SKILL.md or AGENTS.md directly.** If the target is a SKILL.md (has `name:` / `description:` frontmatter), stop and recommend `skill-toolkit`. If the target is an AGENTS.md, audit only R12/R16/R18 (the rules that apply) and note the limitation — the full rubric is CLAUDE.md-specific.

## Rule application notes

- **R01**: `Bash` `wc -l <path>` once per file. Apply the scope's row. Emit evidence as `"178 lines; project scope HIGH tier (201-300)"`.
- **R12** (secrets): scan prose AND code blocks. A token in a code block is still a committed secret. Emit CRITICAL immediately; do not continue other rules until the user is warned.
- **R16** (imports): parse every `@path` occurrence. `Glob` each one. For relative paths, resolve relative to the containing file (not cwd). For `~/`, expand to `$HOME`. Track depth by recursively resolving imports up to 2 hops; emit HIGH if depth 3+ detected.
- **R17** (AGENTS.md bridging): `Glob` for `AGENTS.md` in the same directory as the target. If present, `Grep` for `@AGENTS.md` in the target. If missing, emit HIGH finding with the exact import line to add as the fix.
- **R18** (gitignore for local): if scope is `local`, `Read` `.gitignore` and check if `CLAUDE.local.md` is covered. Use literal match or common glob patterns (`CLAUDE.local.md`, `*.local.md`, `CLAUDE.*.md`).
- **R20** (no frontmatter): check the first non-empty line. If it is `---`, the file has frontmatter — HIGH finding, evidence the first line.
- **R21** (command resolvability): skip if no recognizable config file is present (emit LOW advisory instead of HIGH).

## Anticipated failure modes

Hypothesized — validate against real runs and update.

- **Secret false-positives in regex examples**: if the file intentionally documents what secrets look like (e.g. in a `## Gotchas` section about not committing secrets), the regex will match. Emit finding anyway — user confirms during `improve`.
- **R01 miscounting**: do not count trailing empty lines; use the last non-empty line's number as the total.
- **Profile misdetection in monorepo with ML sub-package**: first-match wins. If the repo is a monorepo with an ML package, the top-level audit uses `monorepo`; sub-package CLAUDE.md audits will re-detect `ml`.
- **R17 false-negative on cross-directory AGENTS.md**: only siblings count. A root `AGENTS.md` next to `packages/foo/CLAUDE.md` is not checked — emit a MEDIUM advisory instead.
- **R15 (hook migration) is not cross-file-aware**: we do not parse `.claude/settings.json` to confirm a hook exists. Emit as advisory only.

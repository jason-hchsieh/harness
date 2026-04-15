# Audit report template

Scaffold for the report emitted by `mode-audit.md`. Substitute the placeholders in `<angle brackets>`.

Report format is markdown so it is readable in the conversation, in a file, and in pull request comments. Severity headings use `[CRITICAL]` / `[HIGH]` / `[MEDIUM]` / `[LOW]` tags rather than emoji so the format is grep-able.

---

```
# CLAUDE.md Audit Report

Target: <path/to/CLAUDE.md>
Date: <UTC ISO-8601 timestamp, e.g. 2026-04-15T12:34:56Z>
Scope: <project | user | local | managed>
Profile: <default | monorepo | ml | open-source | compliance>
Rubric: claude-md-toolkit v0.1.0

## Summary

- CRITICAL: <n> | HIGH: <n> | MEDIUM: <n> | LOW: <n>
- Line count: <n> (budget for <scope>: PASS ≤<x>, WARN ≤<y>, HIGH ≤<z>, CRITICAL >z)
- Headings: <h1-count> h1, <h2-count> h2, <h3-count> h3, <h4+-count> h4+
- Imports (@path): <n> total, max depth <d>
- Emphasis tokens (IMPORTANT/MUST/NEVER/ALWAYS/YOU MUST): <n> (budget ≤5)
- AGENTS.md sibling: <present | absent>
- AGENTS.md bridge (@AGENTS.md import): <present | missing | n/a>
- .gitignore covers CLAUDE.local.md: <yes | no | n/a>
- Frontmatter: <absent (good) | present (R20 HIGH violation)>

## Findings

### [CRITICAL] R<nn> <short rule name>

- **File**: <path>:<line>
- **Evidence**: <verbatim snippet or measurement>
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: <one-line suggestion>

### [HIGH] R<nn> <short rule name>

- **File**: <path>:<line>
- **Evidence**: <verbatim snippet or measurement>
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: <one-line suggestion>

### [MEDIUM] R<nn> <short rule name>

- **File**: <path>:<line>
- **Evidence**: <verbatim snippet or measurement>
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: <one-line suggestion>

### [LOW] R<nn> <short rule name>

- **File**: <path>:<line>
- **Evidence**: <verbatim snippet or measurement>
- **Confidence**: HIGH | MEDIUM | LOW
- **Fix**: <one-line suggestion>

## Next step

To apply these fixes, run:

    claude-md-toolkit improve <path> --report=<path-to-this-report-or-paste-inline>

If the report has zero CRITICAL and zero HIGH findings, the file is in good shape.
```

---

## Ordering rules

1. Group findings by severity: CRITICAL → HIGH → MEDIUM → LOW.
2. Within each severity group, sort by rule id ascending (R01 before R03 before R12).
3. Emit one finding per violation. Exception: R01 (file-level length) emits one finding per file.

## Multi-file sweep (repo-root input)

When `audit` is pointed at a repo root, prepend a sweep summary:

```
# CLAUDE.md Sweep Summary — <repo-name>

Target: <repo-root>
Date: <UTC ISO-8601>
Files discovered: <n> total (CLAUDE.md: <a>, CLAUDE.local.md: <b>)
AGENTS.md files: <c>
Total findings: CRITICAL <n> | HIGH <n> | MEDIUM <n> | LOW <n>

## Per-file summary

| Path                        | Scope   | Profile   | CRIT | HIGH | MED | LOW | Lines |
|-----------------------------|---------|-----------|------|------|-----|-----|-------|
| ./CLAUDE.md                 | project | default   | 0    | 2    | 3   | 1   | 178   |
| packages/api/CLAUDE.md      | project | default   | 0    | 0    | 1   | 0   | 94    |
| ~/.claude/CLAUDE.md         | user    | default   | 0    | 1    | 0   | 0   | 42    |
```

Then the individual per-file reports follow in discovery order.

## Empty-findings case

If the audit finds zero violations, still emit the full report header with all zero counts, then:

```
## Findings

(none — file passes all applicable R01–R22 checks at this rubric version)

## Next step

File passes. No action required. Consider re-auditing when the project evolves (new commands, architectural changes, or scope shift).
```

Do not skip the report just because there are no findings — an empty pass report is a valid audit artifact.

## Report-as-file convention

By default, reports are emitted to the conversation. If the user explicitly asks for a file, write to `.claude/audits/CLAUDE-md-<UTC-date>.md` (create the directory if missing). The path is chosen so audit artifacts live alongside other Claude Code state.

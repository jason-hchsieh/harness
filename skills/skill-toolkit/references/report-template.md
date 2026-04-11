# Audit report template

Scaffold for the report emitted by `mode-audit.md`. Substitute the placeholders in `<angle brackets>`.

Report format is markdown so it is readable in the conversation, in a file, and in pull request comments. Severity headings use `[CRITICAL]` / `[HIGH]` / `[MEDIUM]` / `[LOW]` tags rather than emoji so the format is grep-able.

---

```
# Skill Audit Report — <skill-name>

Target: <path/to/SKILL.md>
Date: <UTC ISO-8601 timestamp, e.g. 2026-04-11T12:34:56Z>
Rubric: harness-toolkit v0.1.0

## Summary

- CRITICAL: <n> | HIGH: <n> | MEDIUM: <n> | LOW: <n>
- Body length: <lines> lines (budget 500 / WARN 600 / HIGH 850)
- Description length: <chars> chars (hard 1024 / soft 250)
- Frontmatter fields present: <list>
- Frontmatter fields missing (recommended): <list>

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

    skill-toolkit improve <path> --report=<path-to-this-report-or-paste-inline>

If the report has zero CRITICAL and zero HIGH findings, the skill is in good shape. Consider running `/reload-plugins` to pick up recent changes, then commit.
```

---

## Ordering rules

1. Group findings by severity in the order CRITICAL → HIGH → MEDIUM → LOW.
2. Within each severity group, sort by rule id ascending (R01 before R03 before R14).
3. Emit one finding per violation — do not collapse multiple R02 violations across multiple files into one finding (the `improve` mode addresses each one individually).

## Multi-skill reports (plugin root input)

When `audit` is pointed at a plugin root and finds multiple `SKILL.md` files, emit a cross-skill summary header first, then one `# Skill Audit Report — <name>` section per skill.

Cross-skill summary block:

```
# Plugin Audit Summary — <plugin-name>

Target: <plugin-root-path>
Skills audited: <n>
Total findings: CRITICAL <n> | HIGH <n> | MEDIUM <n> | LOW <n>

## Per-skill summary

| Skill        | CRIT | HIGH | MED | LOW | Body lines | Desc chars |
|--------------|------|------|-----|-----|------------|------------|
| skill-alpha  | 0    | 1    | 2   | 0   | 247        | 312        |
| skill-beta   | 1    | 0    | 0   | 1   | 512        | 189        |
```

Then the individual per-skill reports follow.

## Empty-findings case

If the audit finds zero violations, still emit the report header and the Summary block with all zeroes, followed by:

```
## Findings

(none — skill passes all R01–R20 checks at this rubric version)

## Next step

Skill passes. Run /reload-plugins if you haven't already, then commit.
```

Do not skip the report just because there are no findings — an empty pass report is a valid audit artifact.

# Mode: audit

Run the R01–R20 rubric against an existing SKILL.md and emit a severity-tagged findings report. **This mode is read-only.**

## Hard preamble — READ THIS FIRST

In this mode you **MUST NOT** invoke `Write` or `Edit`. The only output is the findings report (emitted into the conversation, not written to disk unless the user explicitly asks).

If the user says "fix this" or "apply these fixes" during audit, STOP and tell them to run `skill-toolkit improve <path> --report=<this-report>` instead. Do not apply fixes yourself — the whole point of audit is an independent read-only pass. Mixing audit and improve in the same turn defeats the purpose.

If the audit reveals that files the SKILL.md references are missing, do NOT create them. Report the missing files as an R16 finding instead.

## Input contract

- `$ARGUMENTS` after the mode token is required. It is one of:
  - Path to a `SKILL.md` file (e.g. `.claude/skills/hello/SKILL.md`).
  - Path to a skill directory containing a `SKILL.md` (e.g. `.claude/skills/hello`).
  - Path to a plugin root containing a `skills/` directory — recurse one level and audit each SKILL.md, emitting one report section per skill.
- If the path doesn't exist, STOP and ask the user for a valid path. Do not proceed with a partial audit.

## Phase 1 — Load the target

1. Resolve the input path to a concrete `SKILL.md` file (or a list of them if a plugin root was given).
2. For each file, use `Read` to load the full contents.
3. Parse the frontmatter: find the opening `---` and the closing `---`; everything between is YAML. Extract all fields present.
4. Count the body: lines after the closing `---`, excluding trailing whitespace-only lines from the total.
5. Load `rubric.md` if it is not already in context (the router usually loads it first).

## Phase 2 — Run the checks

Run every applicable rule from R01–R20 against the target. For each violation, create a Finding record with these fields:

- `rule`: the rule id (e.g. `R03`)
- `severity`: `CRITICAL` | `HIGH` | `MEDIUM` | `LOW`
- `confidence`: `HIGH` | `MEDIUM` | `LOW` (per the rubric)
- `location`: `<path>:<line>` pointing to the offending line when possible
- `evidence`: a short verbatim snippet or measurement (e.g. `"description is 1187 chars"`)
- `fix`: a one-line suggestion for how to resolve it

### Frontmatter checks (R01–R13)

Walk every field documented in `frontmatter-schema.md`. For each:

- **R01 `name`**: required; regex `^[a-z0-9][a-z0-9-]{0,63}$`; must equal directory basename. Use `Bash` `basename` or string ops to derive the directory name.
- **R02 `description`**: required; measure char count. ≤1024 hard, ≤250 soft.
- **R03 `description` structure**: grep for "Use when" (case-insensitive). If sibling skills exist in the same plugin, also grep for "Do NOT use" (or "Do not use").
- **R04 `argument-hint`**: if the body contains the literal string `$ARGUMENTS`, check that this field is present.
- **R05 `disable-model-invocation`**: check it is explicitly `true` or `false`. If `true`, grep the body for a rationale line (something mentioning "destructive", "side effect", "manual only", etc.).
- **R06 `user-invocable`**: if `false`, check body prose tone matches reference-material usage (heuristic, LOW confidence).
- **R07 `model`**: if set, must be one of `haiku`/`sonnet`/`opus`. WARN if pinned without nearby justification.
- **R08 `effort`**: if set, must be one of `low`/`medium`/`high`/`max`. WARN if pinned.
- **R09 `allowed-tools`**: must be present; no wildcards; every listed tool must be a real tool name. If `Bash` is listed, grep the body for a one-line justification.
- **R10 `shell`**: if set, must be `bash` or `powershell`.
- **R11 `context` + `agent`**: if `context: fork`, `agent` must be present and non-empty. If `agent` is present but `context` is not `fork`, flag HIGH.
- **R12 `hooks`**: if present, for each hook value use `Glob` to check the referenced path exists. Flag if missing.
- **R13 `paths`**: if present, each entry must be non-empty glob syntax. Flag HIGH if any entry is `*` or `**/*`.

### Body checks (R14–R20)

- **R14 body length**: compare line count to the 500/600/850 thresholds.
- **R15 progressive disclosure**: if body > 400 lines, grep for any `references/` link; if zero, flag HIGH.
- **R16 supporting file integrity**: grep the body for patterns like `references/\S+`, `scripts/\S+`, `assets/\S+`. For each match, `Glob` to check it resolves. Flag each missing file.
- **R17 anti-patterns**: grep for `TODO`, `FIXME`, `XXX`, vague verbs in headings, first-person narration.
- **R18 least surprise**: judgement call. Check that:
  - The description doesn't promise capabilities the body doesn't implement.
  - Read-only-sounding descriptions don't list `Write` or `Edit`.
  - Destructive-sounding descriptions have `disable-model-invocation: true`.
- **R19 trigger non-competition**: if the target is one of several skills in the same plugin, compare descriptions for overlapping top trigger phrases. (Single-skill plugins skip this check.)
- **R20 triggerability**: heuristic check — does the description contain at least one concrete noun a user would say? Flag vague descriptions as LOW.

## Phase 3 — Score and aggregate

- Group findings by severity: CRITICAL → HIGH → MEDIUM → LOW.
- Within each severity, sort by rule id.
- Count findings per severity for the summary block.
- Measure the two key scalars: body line count, description character count.

## Phase 4 — Emit the report

Use the scaffold in `references/report-template.md`. Fill in:

- Target path
- Current UTC timestamp (use `Bash` `date -u +"%Y-%m-%dT%H:%M:%SZ"`)
- Rubric version (`harness-toolkit v0.1.0`)
- Summary counters
- Scalars (body length, description length)
- All findings, formatted per the template
- The "Next step" section pointing to `improve` mode

If the user passed a plugin root, emit one summary header at the top with cross-skill totals, then one section per skill.

**Output the report into the conversation.** Do NOT write it to a file unless the user explicitly requests a file path. (If they do, you are now mutating the filesystem, which is outside the audit contract — confirm before writing.)

## Phase 5 — Handoff

End the report with an explicit next-step line:

> To apply these fixes, run:
> `skill-toolkit improve <path> --report=<paste-this-report-or-save-to-file>`

If the report has zero CRITICAL and zero HIGH findings, congratulate the user and suggest `/reload-plugins` if the skill is new.

## Things you must not do in audit mode

- Do not `Write` or `Edit` any file.
- Do not apply fixes, even "small" ones.
- Do not collapse multiple violations of the same rule into one finding — emit one finding per violation so `improve` can address them individually.
- Do not invent new rule ids. If a check isn't covered by R01–R20, either add the rule to `rubric.md` first (a separate change) or emit it as an advisory under R18.
- Do not skip the confidence field — it is load-bearing for `improve` prioritization.

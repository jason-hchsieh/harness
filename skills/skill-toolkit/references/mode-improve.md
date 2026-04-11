# Mode: improve

Apply audit findings to a SKILL.md. Minimal-diff edits, severity-first, re-audit at the end.

## Input contract

- `$ARGUMENTS` after the mode token is one of:
  - `<path>` — path to a SKILL.md or skill directory. No report provided — you will run the audit logic inline first, then apply findings.
  - `<path> --report=<report-file>` — path to a SKILL.md plus a path to a saved audit report to consume.
  - `<path> --report=<inline>` — the user may paste an audit report inline into the conversation instead of a file path; accept that too.

## Phase 1 — Load findings

1. If the user gave `--report=<file>`, `Read` the file and parse its Findings section.
2. If the user gave `--report=<inline>` or pasted the report into the conversation, parse it directly from the conversation.
3. If no report was given, load `mode-audit.md` and run the full audit workflow against the target first. Treat the resulting report as the findings source.

A parsed finding is a record of: `rule`, `severity`, `confidence`, `location`, `evidence`, `fix`.

## Phase 2 — Prioritize

Sort findings by severity, then by confidence:

1. CRITICAL (HIGH confidence first, then MEDIUM, then LOW)
2. HIGH (HIGH → MEDIUM → LOW)
3. MEDIUM (HIGH → MEDIUM → LOW)
4. LOW (HIGH → MEDIUM → LOW)

LOW-confidence findings are advisory — surface them but confirm with the user before editing.

## Phase 3 — Apply fixes

For each finding, in priority order:

1. `Read` the target SKILL.md to refresh your view of the current state (another fix may have already changed nearby content).
2. Decide the minimal edit that resolves the finding. Examples:
   - **R01** (name doesn't match directory) → `Edit` the `name:` line.
   - **R02** (description too long) → `Edit` the `description:` line to a tighter version that preserves the "Use when…" clause and the trigger words. Do not rewrite for style.
   - **R03** (missing "Use when…") → `Edit` the description to start with "Use when…".
   - **R09** (`Bash` in allowed-tools without justification) → add a one-line justification comment to the body under a "Tools" section, OR remove `Bash` from allowed-tools if it is unused — ask the user which.
   - **R14** (body too long) → this is the hardest case. Move long sections into `skills/<name>/references/<topic>.md` and replace with a link. Ask the user before splitting more than one section per run.
   - **R16** (missing supporting file) → ask the user whether to create a stub or remove the reference. Do not silently create files.
   - **R17** (anti-patterns) → tight `Edit` on the offending line(s).
3. Apply the edit with `Edit` (not `Write`). Prefer `old_string`/`new_string` pairs tight enough to be unambiguous — do not replace entire sections unless that is genuinely the minimal change.
4. If the finding is ambiguous (LOW confidence or the "fix" field in the report is vague), **ask the user** what to do before editing. Do not guess on LOW-confidence findings.

## Phase 4 — Re-audit

After applying all planned fixes:

1. Re-run the audit logic from `mode-audit.md` against the updated SKILL.md.
2. Compare the new findings list to the original. Some findings should be gone; some may have shifted line numbers.
3. If any CRITICAL or HIGH findings remain, report them to the user and offer to iterate.
4. If only MEDIUM/LOW remain, report them and ask whether to stop or keep going.

## Phase 5 — Handoff

Report to the user:

1. What was fixed, grouped by rule id (e.g. "Fixed 2 R02 (description length), 1 R14 (body length by splitting `references/examples.md`)").
2. What remains, if anything.
3. A reminder to run `/reload-plugins` so the edited skill is picked up by Claude Code.
4. If the skill now passes entirely, suggest committing.

## Things you must not do in improve mode

- Do not rewrite the skill for style. Your only mandate is to resolve the findings the audit report identified. Stylistic rewrites out of scope.
- Do not delete supporting files referenced by the body unless the user explicitly says so.
- Do not apply LOW-confidence findings without user confirmation.
- Do not replace `Edit` with `Write` (a full-file rewrite) except as a last resort when the changes are larger than 30% of the file AND the user confirms.
- Do not loop forever on re-audit. Cap at 3 re-audit iterations per invocation; if findings persist, report and stop.

# Mode: improve

Apply audit findings to a SKILL.md. Minimal diffs, severity-first, re-audit when done.

## Goal

Resolve CRITICAL and HIGH findings from an audit report. Leave MEDIUM/LOW for user decision. Re-audit after fixes to verify and catch regressions.

## Constraints

- **Minimal diffs only.** Use `Edit` (old_string/new_string), not `Write` (full rewrite). Your mandate is to resolve the findings — not to restyle the skill.
- **LOW-confidence findings require confirmation.** Ask the user before editing anything rated LOW confidence.
- **Don't create or delete files silently.** If a finding says a referenced file is missing (R16), ask the user: create a stub, or remove the reference?
- **Cap at 3 re-audit iterations.** If findings persist after 3 rounds, report and stop.

## Input

`$ARGUMENTS` after `improve` is:
- `<path>` — SKILL.md or skill directory. No report → run audit inline first, then apply.
- `<path> --report=<file>` — consume a saved audit report.
- `<path> --report=<inline>` — user may paste the report into conversation.

A finding is: `rule`, `severity`, `confidence`, `location`, `evidence`, `fix`.

## How it works

1. **Get findings** — from report file, inline paste, or run audit.
2. **Sort by severity+confidence** — CRITICAL/HIGH first, then MEDIUM/LOW. Within same severity, HIGH confidence before LOW confidence.
3. **Apply each fix** — minimal `Edit`. Re-read the file between edits (prior fixes shift content).
4. **Re-audit** — run `rubric.md` checks against the updated file. Report what was fixed and what remains.
5. **If CRITICAL/HIGH persist** — offer to iterate (up to 3 total rounds).

## Handoff

Report to user: what was fixed (grouped by rule id), what remains, `/reload-plugins` reminder.

## Gotchas

Claude tends to fail on these:

- **Over-editing.** You'll be tempted to clean up surrounding code while you're fixing a finding. Don't. Only touch what the finding identifies.
- **R14 (body too long) is the hardest fix.** You can't just delete lines. Move long sections into `references/` files and replace with a link. Ask the user before splitting more than one section.
- **R02 description rewrites lose trigger words.** When tightening a description, preserve the "Use when…" clause and the concrete nouns users would say. Tightening for length must not kill triggerability.
- **Stale line numbers.** After applying one fix, the line numbers from the report are wrong. Always re-read the file before the next edit.
- **Don't guess on ambiguous R09 fixes.** If `Bash` is in allowed-tools but the body doesn't use it, ask the user: remove Bash, or add a justification?

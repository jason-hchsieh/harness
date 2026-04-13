# Mode: improve

Apply audit findings to a SKILL.md. Minimal diffs, severity-first, re-audit when done.

## Goal

Resolve CRITICAL and HIGH findings from an audit report. Leave MEDIUM/LOW for user decision. Re-audit after fixes to verify and catch regressions.

## Input

`$ARGUMENTS` after `improve` is:
- `<path>` — SKILL.md or skill directory. No report → run audit inline first, then apply.
- `<path> --report=<file>` — consume a saved audit report.
- `<path> --report=<inline>` — user may paste the report into conversation.

## Specifications

### Fix patterns by rule

Common fixes keyed to specific rules. Use these as defaults; ask the user when ambiguous.

| Rule | Fix pattern |
|------|------------|
| R01 (name mismatch) | `Edit` the `name:` line to match the directory basename. |
| R02 (description too long) | Tighten the `description:` line. Preserve "Use when…" and trigger nouns. Don't rewrite for style. |
| R03 (missing "Use when…") | Prepend "Use when…" to the existing description. |
| R05 (missing disable-model-invocation) | Add `disable-model-invocation: false` (or `true` with rationale if destructive). |
| R09 (Bash without justification) | Ask user: add a justification comment to the body, OR remove `Bash` from allowed-tools. |
| R14 (body too long) | Move long sections into `references/` files and replace with a link. Ask before splitting >1 section. |
| R16 (missing file) | Ask user: create a stub file, OR remove the dangling reference from the body. |
| R17 (anti-pattern) | Tight `Edit` on the offending line — remove TODO markers, vague verbs, first-person narration. |

For rules not listed above, derive the minimal edit from the finding's `fix` suggestion and the rule definition in `rubric.md`.

### Handoff format

Report to the user:
- What was fixed, grouped by rule id (e.g. "Fixed 2x R02, 1x R14 by splitting `references/examples.md`")
- What remains (severity + rule id)
- `/reload-plugins` reminder

## Constraints

- **Minimal diffs only.** Use `Edit` (old_string/new_string), not `Write` (full rewrite). Only touch what the finding identifies.
- **LOW-confidence findings require user confirmation.** Don't guess on advisory findings.
- **Don't create or delete files silently.** Always ask on R16 fixes.
- **Cap at 3 re-audit iterations.** If findings persist after 3 rounds, report and stop.
- **Re-read between edits.** Prior fixes shift content — stale line numbers cause wrong edits.

## Workflow

This mode is an iterative loop, not a linear pipeline:

1. Get findings (from report or inline audit).
2. Sort by severity+confidence (CRITICAL/HIGH first).
3. Apply minimal edits — re-read the file between each.
4. Re-audit the updated file.
5. If CRITICAL/HIGH persist → iterate (up to 3 rounds).

## Anticipated failure modes

These are hypothesized — not yet validated by real runs. Update as actual failures are discovered.

- **Over-editing.** Fixing a finding tempts you to clean up surrounding code. Don't — only touch what the finding identifies.
- **R14 is the hardest fix.** You can't just delete lines. Ask before splitting sections.
- **R02 rewrites lose trigger words.** Tightening a description for length must not kill triggerability.
- **Stale line numbers after prior edits.** Always re-read before each Edit.
- **Ambiguous R09.** If Bash is listed but the body doesn't use it, don't silently remove it — ask.

# Mode: improve

Consume an audit report and apply minimal-diff fixes to CLAUDE.md, then re-audit.

## Goal

Translate each audit finding into an `Edit` (or rarely, a `Write`) that resolves exactly that finding — no stylistic rewrites, no refactors beyond the finding's scope. After applying fixes, re-run audit and emit the residual.

## Input

`$ARGUMENTS` after `improve` is `<path> [--report=<file-or-inline>]`.

- If `--report=` is omitted: run the audit inline first (load `mode-audit.md`), then proceed with that report.
- If `--report=<path>`: `Read` the report file.
- If `--report=` contains the report inline (e.g. pasted content wrapped in triple backticks), parse it directly.

If the target path or report is missing, ask the user.

## Fix ordering

1. **CRITICAL** findings first, one at a time, each followed by a mental verification that the fix does not regress another rule.
2. **HIGH** findings next.
3. **MEDIUM** findings next.
4. **LOW** findings: ask the user whether to apply. They are often stylistic (R05 horizontal rules, R09 imperative voice on non-critical lines).

Within a severity tier, apply in rule-id order (R01 before R03 before R12…). This keeps the diff predictable when multiple findings interact (e.g. R01 length reduction may also resolve R04 oversized sections).

## Fix patterns by rule

### R01 (length over budget)

If the file exceeds the scope's budget, the fix is **extraction**, not compression:

1. Identify the 2–3 largest sections (by line count).
2. Suggest extracting to `@docs/<topic>.md` or `.claude/rules/<topic>.md`.
3. Replace the section body with a one-line summary plus the import.
4. Ask the user to confirm the extraction target path before writing.

Do NOT delete content outright unless the finding explicitly marks a section as duplicative (R10) or off-topic (R07).

### R02 (heading hierarchy)

- `h4+` headings: promote or demote to `h3`, or restructure.
- Multiple `h1`: demote all but the first to `h2`.

### R03 (critical rules not front-loaded)

Move the section containing `IMPORTANT` / `MUST` / `NEVER` to appear before `Commands`. Preserve its internal structure.

### R04 (section exceeds 30 lines)

Same extraction pattern as R01 — pull the oversized section into `@docs/<topic>.md`.

### R05 (horizontal rules)

Delete `---` decoration lines. Preserve at most one, only if it delimits the References appendix.

### R06 (long bullets)

Split the bullet into multiple or demote into a sub-section. Do not silently truncate.

### R07 (deletability — advisory)

Delete the line. If the line carries context the user cares about, ask first.

### R08 (vague phrases)

Replace each match with a concrete assertion. Examples:
- `"Write clean code"` → ask: "What specific check would fail if this rule is broken?"
- `"Handle errors gracefully"` → `"Wrap async operations in try/catch; log errors via `logger.error(err, {ctx})`"`.

If the user can't articulate a concrete rule, delete the line (R07).

### R09 (non-imperative voice)

Rewrite sentence-by-sentence: `"We use 2 spaces"` → `"Use 2-space indentation"`. Keep the facts, change the mood.

### R10 (codebase duplication)

Delete the offending block. Replace with a one-line pointer to code (`"See `src/api/` for handler layout"`) only if the pointer itself has value.

### R11 (embedded tutorial / API docs)

Extract to a linked doc. Replace the prose with a one-line import (`@docs/api-overview.md`).

### R12 (secret)

CRITICAL. Remove the secret text immediately. Then:
1. Warn the user verbally and in the improve output.
2. Suggest rotating the secret.
3. If the file is tracked in git, recommend `git filter-repo` / BFG to scrub history.
4. Do NOT move the secret to CLAUDE.local.md or .env as part of the edit — that's the user's call after rotation.

### R13 (emphasis budget)

List every emphasis token and its line. Ask the user which ≤5 to keep. Demote the rest (remove `IMPORTANT:` prefix, convert `YOU MUST` to `Use` / `Avoid` imperatives).

### R14 (self-contradiction)

Present the contradiction to the user and ask which rule is authoritative. Apply the user's choice — never resolve silently.

### R15 (hook migration advisory)

Do NOT edit `.claude/settings.json` from improve mode. Instead:
1. In the CLAUDE.md, soften the rule or move it to a new `## Hooks` section that cross-references the intended `settings.json` entry.
2. Output a suggested `settings.json` hook block in the improve result so the user can apply it separately.

### R16 (broken import)

- If the path is wrong but the file exists nearby, fix the path.
- If the file doesn't exist, ask the user whether to (a) create the target file, or (b) remove the import. Do NOT silently delete.

### R17 (AGENTS.md not bridged)

Insert `@AGENTS.md` at the top of the file (before the first `#` heading, or as the first line if no title). Add a blank line after. If significant Claude-specific content already exists, leave it in place below the import.

### R18 (scope leakage)

- Gitignore miss: suggest an `.gitignore` edit in the improve output; do NOT edit `.gitignore` yourself unless the user confirms.
- Personal content in project file: move the offending lines to a `CLAUDE.local.md` (create if missing, with gitignore entry). Ask the user first.

### R19 (HTML comment with Claude-facing instructions)

Move the instruction out of the HTML comment into regular prose. Preserve maintainer-only notes inside the comment.

### R20 (frontmatter present)

Delete the entire frontmatter block (everything from the leading `---` to the closing `---` inclusive, plus the following blank line). CLAUDE.md is pure markdown.

### R21 (unresolvable command)

Ask the user: is the command wrong, or is the script missing from `package.json` / `Makefile`? Fix whichever the user confirms — but improve mode does NOT edit `package.json` / `Makefile` (that's a code change, not a CLAUDE.md change).

### R22 (version-aware features)

Add a one-line note near the top: `<!-- Requires Claude Code v2.1+ for HTML comment stripping and .claude/rules path scoping -->`. Do not change the feature usage itself.

## Re-audit protocol

After applying all in-scope fixes:

1. Re-load `mode-audit.md` and re-run the audit.
2. Compare the new findings set to the original.
3. Emit a residual report:

```
## Improve Residual

Applied: <n> fixes (CRIT <n>, HIGH <n>, MED <n>, LOW <n>)
Remaining findings: CRITICAL <n> | HIGH <n> | MEDIUM <n> | LOW <n>

### Carried over
[list of findings that persisted — usually LOW that the user deferred, or advisory items]

### Newly introduced
[findings that appeared as a side effect of fixes — investigate these, they indicate a fix regression]
```

If any "newly introduced" findings exist, stop and surface them to the user before claiming success.

## Constraints

- **Minimal diff.** Do not reflow paragraphs, reorder sections beyond what R03 requires, or change voice on non-flagged lines.
- **No stylistic rewrites.** If a line passes the rubric, leave it alone even if you personally would write it differently.
- **Confirm before cross-file edits.** `.gitignore`, `settings.json`, extracting to new files — always confirm the target path.
- **Never edit the audit report.** The report is an input, not a mutable artifact.
- **Do not silently resolve contradictions (R14).** Ask.
- **Do not rotate secrets (R12).** Only remove the text; rotation is the user's responsibility.

## Anticipated failure modes

- **Cascading R01 fixes**: extracting one section may still leave the file over budget. Apply extraction, re-measure, extract again if needed. Do not shoe-horn multiple extractions into one edit — one per confirmed target path.
- **R17 conflicting with existing top content**: if the file already starts with `# Project Name` and narrative prose, inserting `@AGENTS.md` above it is fine but may feel abrupt. That's acceptable — the import is semantically first.
- **R08 replacements may reintroduce R08**: when replacing a vague phrase, be careful not to swap one for another (`"Write good tests"` → `"Write proper tests"` is not a fix).
- **R13 demotion may remove user-intended emphasis**: always ask which ≤5 to keep.

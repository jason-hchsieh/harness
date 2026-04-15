# Mode: create

Scaffold a new CLAUDE.md tuned to the selected scope and profile, then self-check against the rubric.

## Goal

Produce a working CLAUDE.md at the correct path that passes R01–R22 at creation time, with fields filled in from evidence in the repo (not guesses).

## Input

`$ARGUMENTS` after `create` is an optional scope: one of `project` (default), `user`, `local`, `managed`.

## Target path by scope

| Scope    | Target path                                      | Git behavior           |
|----------|--------------------------------------------------|------------------------|
| project  | `./CLAUDE.md`                                    | Commit to shared repo  |
| user     | `~/.claude/CLAUDE.md`                            | Personal dotfiles only |
| local    | `./CLAUDE.local.md`                              | Must be gitignored     |
| managed  | OS-specific (see `scope-matrix.md`)              | MDM-deployed           |

If the target file already exists, STOP and ask the user whether to:
1. Audit the existing file instead (`claude-md-toolkit audit <path>`), or
2. Overwrite (requires explicit confirmation and shows a diff preview), or
3. Create at a different scope.

Do not silently overwrite.

## Workflow

### Step 1 — Path & scope sanity

- Confirm the target path from the scope argument.
- For `local`: check `.gitignore` now. If `CLAUDE.local.md` is not covered, surface this to the user BEFORE writing (R18 pre-check). Offer to append `CLAUDE.local.md` to `.gitignore` with confirmation.
- For `user`: confirm `~/.claude/` directory exists; create if missing.
- For `managed`: warn that individual writes rarely belong here — managed policy is deployed via MDM / Ansible / Group Policy. Confirm the user really wants this scope.

### Step 2 — Profile detection

Use the same heuristic as `mode-audit.md`:
- `package.json` with `workspaces`, `pnpm-workspace.yaml`, `turbo.json`, or `lerna.json` → `monorepo`
- ML-adjacent dependencies (torch / tensorflow / transformers / wandb / mlflow) or model/dataset directories → `ml`
- `CONTRIBUTING.md` + `LICENSE` + public-looking `README.md` → `open-source`
- `SECURITY.md` with HIPAA/SOC2/PCI/GDPR keywords or `compliance/` dir → `compliance`
- Otherwise → `default`

Confirm the detected profile with the user. Load `platform-profiles.md` only if the profile is non-default.

### Step 3 — AGENTS.md detection

`Glob` for a sibling `AGENTS.md`. If present:
- Load `agents-md-bridge.md` for guidance.
- The generated CLAUDE.md will start with `@AGENTS.md` and contain only Claude-specific additions.
- Skip intent-capture questions already answered by AGENTS.md (read it first to avoid duplication).

### Step 4 — Intent capture (up to 5 questions)

Ask only the questions that can't be answered from repo evidence. In priority order:

1. **Critical rules**: "Any hard rules (`IMPORTANT` / `NEVER`) Claude must never violate? Answer in ≤3 bullets or 'none'."
2. **Non-standard commands**: "Any build/test/lint commands that differ from the default `npm/pnpm/make/poetry` conventions? (We'll detect the standard ones automatically.)"
3. **Module boundaries**: "Any import rules or architectural boundaries Claude must respect? (e.g. 'handlers/ may not import db/')"
4. **Gotchas**: "Any non-obvious traps a new contributor would hit? (e.g. required env vars, local service dependencies, version pins)"
5. **Git/PR conventions**: "Any non-default branch naming, commit format, or PR checklist?"

Skip questions when the answer is already evident. If the user answers "none" to all applicable questions AND the repo has no detectable non-default commands, inform the user a CLAUDE.md may be unnecessary at this stage — offer to scaffold the minimum anyway or abort.

### Step 5 — Repo evidence extraction

Auto-fill from repo scan (do NOT ask the user what the tooling is):

- **Commands**: read `package.json.scripts`, `Makefile` targets, `pyproject.toml [tool.poetry.scripts]` / `[project.scripts]`, `Cargo.toml` if present. Pick the 3–5 most likely daily commands: install, dev, test (single-file invocation if possible), lint, build.
- **Project structure**: enumerate top-level `src/` subdirectories or workspace packages. One line each, ≤6 entries. If structure is obvious from a quick tree, skip this section.
- **Testing framework**: detect from dependencies (jest / vitest / pytest / go test / cargo test).
- **Language-specific style surprises**: ESM vs CJS from `"type"` field; Python formatter from `ruff.toml` / `.flake8`; TypeScript strict from `tsconfig.json`.

### Step 6 — Scaffold from skeleton

1. `Read` `assets/skeleton.md`.
2. `Write` to the target path with sections populated from steps 4–5.
3. Omit sections where there's nothing real to say (an empty "Gotchas" section is worse than no section).
4. For AGENTS.md-bridged files, the top is `@AGENTS.md` followed by a `## Claude Code specific` section. Do not re-state content that AGENTS.md already covers.
5. Keep the file under the scope's PASS budget (project: ≤150 lines; user: ≤30; local: ≤60; managed: ≤80).

### Step 7 — Self-check

Run a subset of the rubric against the new file (full `mode-audit.md` is overkill post-create; apply these rules):

- **R01**: measure line count, ensure PASS tier for scope.
- **R02**: confirm heading hierarchy (≤1 h1, no h4+).
- **R12**: scan for secret patterns (should be zero since we just wrote it, but verify).
- **R13**: count emphasis tokens ≤5.
- **R18**: if scope is `local`, confirm `.gitignore` is settled.
- **R20**: verify no frontmatter was accidentally added.
- **R21**: for commands in `## Commands`, verify they resolve against the detected config files.

If any self-check fails, fix in place before returning — do not leave the user with a file that immediately fails audit.

### Step 8 — Next steps

Output a short next-steps block:
- "Run `claude-md-toolkit audit <path>` to independently verify." (always)
- "Add `CLAUDE.local.md` to `.gitignore`" (only for local scope when not already set)
- "Commit `<path>` so your team can contribute to it." (only for project scope)
- "Restart Claude Code to pick up the new memory file." (always)

## Constraints

- **Never write outside the target path** without explicit confirmation. No speculative extraction to `.claude/rules/` during create — that's a post-create refactor.
- **Respect existing files.** If `CLAUDE.md` exists, never overwrite without confirmation.
- **Do not guess commands.** If the test runner is unclear, ask — don't hallucinate `npm test`.
- **Do not emit frontmatter.** R20. CLAUDE.md is pure markdown.
- **No more than 5 intent questions.** If the repo answers most of them, ask fewer. Respect the user's time.
- **Do not pad to hit a length.** Under 60 lines is fine if that's all there is to say. Short is good.

## Anticipated failure modes

- **Over-asking when repo is evident**: if the repo has `package.json` with clear scripts, don't ask about build commands. Extract silently.
- **Scaffolding with placeholder `<cmd>` left behind**: the skeleton uses `<angle brackets>` for placeholders. Verify none remain in the written file.
- **Profile misdetection in polyglot repos**: ask the user to confirm the profile instead of silently misapplying.
- **AGENTS.md out of sync**: if AGENTS.md exists but is clearly stale / incomplete, don't bridge blindly. Surface the situation and let the user decide to bridge, fork, or regenerate.
- **Managed scope misuse**: if the user asks for managed scope on a laptop without MDM context, confirm the intent — they may actually want `user` scope.

# Scope matrix

Decision table for which scope a CLAUDE.md belongs in, what its budget is, and how to handle each. Used by `mode-create.md` (scope selection) and `mode-audit.md` (R01 length budgets and R18 gitignore checks).

## The four scopes

| Scope     | Target path                                                                                                                  | Git behavior                  | R01 PASS budget | Purpose                                                                                 |
|-----------|------------------------------------------------------------------------------------------------------------------------------|-------------------------------|-----------------|-----------------------------------------------------------------------------------------|
| project   | `./CLAUDE.md` or `./.claude/CLAUDE.md`                                                                                        | Commit to shared repo         | ≤150 lines      | Team-shared: architecture, build/test commands, conventions, gotchas                    |
| user      | `~/.claude/CLAUDE.md`                                                                                                         | Personal dotfiles repo only   | ≤30 lines       | Cross-project personal preferences (shell, verbosity, tone preferences)                 |
| local     | `./CLAUDE.local.md`                                                                                                           | MUST be gitignored            | ≤60 lines       | Per-project personal notes (sandbox URLs, local test data, branch-specific scratch)     |
| managed   | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux/WSL: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | MDM / Ansible / Group Policy  | ≤80 lines       | Organization-wide compliance baseline, cannot be excluded by individual users           |

## Load order & merge behavior

All discovered CLAUDE.md files are **concatenated** into the session, not overridden. The merge order is:

1. Managed policy (cannot be excluded)
2. User (`~/.claude/CLAUDE.md`)
3. Project, walking the directory tree from cwd up to repo root
4. Subdirectory CLAUDE.md files are lazy-loaded when Claude reads files in that directory
5. Within a single directory, `CLAUDE.md` is loaded before `CLAUDE.local.md` — so local content is the "last word" and wins conflicts

## Scope decision algorithm (for `mode-create.md`)

```
1. Does this instruction apply to every project I work on?
   → user scope (~/.claude/CLAUDE.md)

2. Does this instruction apply to this project only, AND should teammates see it?
   → project scope (./CLAUDE.md)

3. Does this instruction apply to this project only, AND is it personal (sandbox URLs, my preferences, my local env)?
   → local scope (./CLAUDE.local.md)  — and ensure .gitignore covers it

4. Does this instruction apply organization-wide and need to be mandatory for every developer?
   → managed scope — but deploy via MDM / Ansible, not via `claude-md-toolkit create`.
     This mode only produces the file; distribution is out of scope.

5. Multiple answers yes?
   → split across multiple scopes. A single rule may appear in user OR project, never both.
```

## Common misplacement patterns

| Misplacement                                                                    | Correct home                   | Flagged by |
|---------------------------------------------------------------------------------|--------------------------------|------------|
| Personal shell preference in project CLAUDE.md                                  | `~/.claude/CLAUDE.md`          | R18        |
| Sandbox URL with developer's name in project CLAUDE.md                          | `./CLAUDE.local.md`            | R18        |
| Cross-project coding style in `./CLAUDE.md` of every repo                       | `~/.claude/CLAUDE.md` once     | Advisory   |
| Compliance rule in user CLAUDE.md                                               | managed scope                  | Advisory   |
| API keys anywhere                                                               | `.env` + `.gitignore`          | R12        |
| Build command in user CLAUDE.md                                                 | project CLAUDE.md              | Advisory   |

## `.gitignore` patterns for local scope

When creating `CLAUDE.local.md`, ensure `.gitignore` contains one of:

```
CLAUDE.local.md
```

or a broader pattern that covers it:

```
*.local.md
CLAUDE.*.md
.claude/local/
```

`mode-create.md` and `mode-audit.md` both check for one of these patterns literally before flagging R18.

## Managed scope notes

The router's `mode-create.md` can write to managed paths if the user explicitly requests it, but should warn:

1. Managed CLAUDE.md is typically deployed by IT via MDM (Jamf, Intune, Workspace ONE), Ansible, Chef, or Group Policy — not by individual `claude-md-toolkit create` invocations.
2. Writing directly to `/etc/claude-code/CLAUDE.md` usually requires root/admin permissions; the write may fail.
3. Individual users cannot exclude managed policy with `claudeMdExcludes` — it is always loaded. Use it sparingly.
4. Managed content should be small (≤80 lines), high-signal, and change rarely. Treat it like a corporate policy doc.

Appropriate managed content:
- "All code must follow SOC2 logging conventions (see @corp/security/logging.md)."
- "Do not include customer PII in test fixtures."
- "Production access requires PR approval from #sre."

Inappropriate managed content:
- Build commands (belongs in project scope).
- Personal preferences (belongs in user scope).
- Frequently changing policies (will bit-rot; use a central doc system instead).

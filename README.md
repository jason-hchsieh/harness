# harness-toolkit

A Claude Code plugin that helps you **create**, **audit**, and **improve** Claude Agent Skills.

One omni skill, three modes, one rubric. Ships as a single `skill-toolkit` skill whose body is a thin router; each mode's real instructions live under `skills/skill-toolkit/references/` and are loaded only when that mode runs (progressive disclosure).

## What it does

| Mode | Purpose | Writes files? |
|------|---------|---------------|
| `create` | Scaffold a new SKILL.md under `.claude/skills/<name>/`, walk through intent capture, self-check against the rubric | Yes |
| `audit` | Run a 20-rule rubric against an existing SKILL.md and emit a severity-tagged findings report | No (read-only) |
| `improve` | Consume an audit report and apply minimal-diff fixes, then re-audit | Yes |

All three modes share the same validation rubric (`references/rubric.md`) and the same frontmatter spec (`references/frontmatter-schema.md`), so create/audit/improve can never disagree about what "valid" means.

## The rubric at a glance

Twenty rules (`R01`–`R20`) covering:

- All **13 official frontmatter fields** (`name`, `description`, `argument-hint`, `disable-model-invocation`, `user-invocable`, `model`, `effort`, `allowed-tools`, `shell`, `context`, `agent`, `hooks`, `paths`)
- Body-level concerns: length budget, progressive disclosure, supporting file integrity, anti-patterns, principle of least surprise, trigger non-competition, triggerability

See `skills/skill-toolkit/references/rubric.md` for the full text.

## Installation (local dev)

```bash
git clone <this-repo> harness
cd harness
claude --plugin-dir .
```

Inside Claude Code:

```
/reload-plugins
```

You should see `skill-toolkit` listed under the `harness-toolkit` plugin. Invoke it via:

- Auto-trigger: just say `"help me create a skill for X"` or `"audit this SKILL.md"` — Claude matches your intent against the description.
- Explicit: `/harness-toolkit:skill-toolkit create <name>` (or `audit`/`improve`).

## Usage examples

### Create a new skill

```
/harness-toolkit:skill-toolkit create my-new-skill
```

The skill asks up to 5 intent questions, scaffolds `.claude/skills/my-new-skill/SKILL.md` from `assets/skeleton.md`, fills the frontmatter, self-checks against the rubric, and reminds you to `/reload-plugins`.

### Audit an existing skill

```
/harness-toolkit:skill-toolkit audit .claude/skills/my-new-skill/SKILL.md
```

Emits a markdown report grouped by severity. Audit mode is strictly read-only — it will never edit your files.

### Apply audit fixes

```
/harness-toolkit:skill-toolkit improve .claude/skills/my-new-skill/SKILL.md --report=<paste or path>
```

Applies minimal `Edit` diffs keyed to the audit findings, re-audits, reports residuals.

## Why a Skill (not a Subagent)?

Short version: Anthropic's own [`skill-creator`](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) is a Skill, not a Subagent. When the authors of both primitives picked one for this exact use case, that's the tiebreaker. Long version is in the plan file at `/root/.claude/plans/vast-drifting-harp.md` — seven ordered reasons covering file mutation, the audit→improve loop, auto-trigger, progressive disclosure, community consensus, and the escape hatch (`context: fork` preserves subagent benefits inside a Skill if you need isolation).

## Directory layout

```
harness/
├── LICENSE
├── README.md                                # this file
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── skill-toolkit/
        ├── SKILL.md                         # thin router
        ├── references/
        │   ├── rubric.md                    # R01–R20 (the single source of truth)
        │   ├── frontmatter-schema.md        # per-field spec (all 13 fields)
        │   ├── mode-create.md               # create workflow
        │   ├── mode-audit.md                # audit workflow (read-only)
        │   ├── mode-improve.md              # improve workflow
        │   ├── anti-patterns.md             # bad-vs-good examples
        │   └── report-template.md           # audit report scaffold
        └── assets/
            ├── skeleton.md                  # blank SKILL.md template
            └── example-skill.md             # minimal rubric-passing example
```

## Credits

- Anthropic's [`skill-creator`](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) — reference implementation this toolkit is modeled on.
- [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) — source for the 13-field frontmatter spec and the "Tips — Skills" guide that the rubric incorporates.
- Community skill-meta-tools that informed the audit design: `nyosegawa/skill-auditor`, `affaan-m/everything-claude-code/skill-stocktake`, `obra/superpowers/writing-skills`.

## License

See `LICENSE`.

# SKILL.md anti-patterns

Concrete bad-vs-good pairs for R17 and related rules. Use this when auditing (to recognize the pattern) and when creating (to avoid writing one in the first place).

---

## 1. Vague description

Fails **R03** (missing "Use when…"), **R20** (no concrete nouns), and often **R18** (principle of least surprise because it promises everything and delivers nothing).

**Bad**:

```yaml
description: Handles various skill-related workflows and automates processes.
```

**Good**:

```yaml
description: Use when the user wants to create a new Claude Agent Skill under .claude/skills/ with valid frontmatter, progressive disclosure, and a rubric-passing body.
```

The good version tells Claude exactly when to fire (user wants to create a skill), what output to produce (files under `.claude/skills/`), and mentions concrete anchors (frontmatter, rubric) the user might say.

---

## 2. Description that competes with a sibling skill

Fails **R19** (trigger non-competition).

**Bad** — two skills in the same plugin:

```yaml
# skills/alpha/SKILL.md
description: Use when the user wants to manage skills.

# skills/beta/SKILL.md
description: Use when the user wants to work with skills.
```

Both descriptions share "manage/work with skills" as the top trigger and neither disambiguates. Claude will flip a coin.

**Good**:

```yaml
# skills/alpha/SKILL.md
description: Use when the user wants to CREATE a new skill from scratch under .claude/skills/. Do NOT use for auditing or improving existing skills — use beta for that.

# skills/beta/SKILL.md
description: Use when the user wants to AUDIT or IMPROVE an existing SKILL.md against the quality rubric. Do NOT use for creating new skills — use alpha for that.
```

---

## 3. Body that inlines 800 lines of reference material

Fails **R14** (body length budget) and **R15** (progressive disclosure).

**Bad**: a single SKILL.md containing the full validation rubric, the full frontmatter schema, 30 anti-pattern examples, and the audit report template — all inline, 900 lines.

**Good**: SKILL.md is a ~200-line router. The rubric lives in `references/rubric.md`. The schema lives in `references/frontmatter-schema.md`. The anti-patterns live in `references/anti-patterns.md`. The report template lives in `references/report-template.md`. The body links to each one and instructs Claude to `Read` them only when needed.

---

## 4. `allowed-tools: *`

Fails **R09** (minimality + wildcards forbidden).

**Bad**:

```yaml
allowed-tools: "*"
```

**Good**:

```yaml
allowed-tools: Read, Write, Edit, Glob, Grep
```

The `*` wildcard isn't a real value — real Claude Code skills enumerate the tools they use. The wildcard pattern usually means the author didn't think about the tool set.

---

## 5. `Bash` in allowed-tools with no justification

Fails **R09** (Bash justification).

**Bad**:

```yaml
allowed-tools: Read, Bash
```

…with no explanation anywhere in the body of what `Bash` is for.

**Good**:

```yaml
allowed-tools: Read, Bash
```

```markdown
## Tools

This skill uses `Bash` for a single purpose: running `date -u` to timestamp audit reports. No other shell usage.
```

The justification lets the audit skill (and a reviewing human) verify that `Bash` access is being used minimally and not as a blank check for arbitrary commands.

---

## 6. Destructive skill without `disable-model-invocation: true`

Fails **R18** (least surprise). If a skill deletes files, pushes to remote, deploys, or mutates external state, it MUST require manual invocation.

**Bad**:

```yaml
name: deploy-production
description: Use when the user wants to deploy the current branch to production.
allowed-tools: Bash
disable-model-invocation: false
```

This will auto-trigger on any user phrasing that mentions deploy — including accidental ones ("what would a deploy look like?").

**Good**:

```yaml
name: deploy-production
description: Use when the user wants to deploy the current branch to production. Requires manual invocation — destructive side effects.
allowed-tools: Bash
disable-model-invocation: true
```

```markdown
## Why auto-invocation is disabled

This skill pushes to the production target and cannot be rolled back. The user must type `/deploy-production` explicitly; Claude will not invoke it from phrasing alone.
```

---

## 7. First-person narration in the body

Fails **R17** (anti-pattern).

**Bad**:

```markdown
## How it works

I will read the SKILL.md and then I'll check the frontmatter fields. After that my plan is to emit a report.
```

**Good**:

```markdown
## How it works

1. Read the target SKILL.md.
2. Check every frontmatter field against `frontmatter-schema.md`.
3. Emit findings using `report-template.md`.
```

Imperative, step-numbered, no "I will". The model is the one following the instructions — writing "I will" is confusing and adds noise.

---

## 8. Empty heading sections

Fails **R17**.

**Bad**:

```markdown
## Phase 3 — Apply fixes

## Phase 4 — Handoff
```

Phase 3 is an empty section. Either remove the heading or fill it in.

**Good**: every `##` heading has at least a sentence of prose or a bulleted list underneath it.

---

## 9. Vague verbs in headings

Fails **R17**. Headings like "Handle things", "Manage state", "Process the input" don't tell the reader or Claude what to actually do.

**Bad**:

```markdown
## Handle the input
## Manage the output
## Process the findings
```

**Good**:

```markdown
## Parse $ARGUMENTS into (mode, target, optional report)
## Write findings to the conversation, not to disk
## Sort findings by severity then by confidence
```

---

## 10. `agent` set without `context: fork`

Fails **R11**.

**Bad**:

```yaml
agent: Explore
```

Without `context: fork`, the `agent` field has no effect — the skill runs inline regardless. This is almost always a misunderstanding of how the two fields interact.

**Good**:

```yaml
context: fork
agent: Explore
```

…or just omit both if the skill should run inline.

---

## 11. Dangling supporting-file references

Fails **R16**.

**Bad**: the body says "See `references/examples.md` for examples" but `references/examples.md` does not exist in the skill directory.

**Good**: every path mentioned in the body resolves. Either create the file (even as a stub) or remove the reference.

---

## 12. Overly broad `paths` glob

Fails **R13**.

**Bad**:

```yaml
paths:
  - "**/*"
```

This auto-activates the skill on every file the user touches. It will drown out every other `paths`-triggered skill and pollute the user's context window.

**Good**:

```yaml
paths:
  - "skills/**/SKILL.md"
  - ".claude/skills/**/SKILL.md"
```

Specific, targeted, fires only when the user is actually working on skill files.

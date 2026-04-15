# claude-md-toolkit rubric (R01–R22)

The single source of truth for validation rules used by `create`, `audit`, and `improve` modes. Every audit finding MUST cite a rule id from this file plus an evidence snippet plus a confidence level.

This rubric targets **CLAUDE.md** (and the equivalent `AGENTS.md` bridging pattern). It does NOT target `SKILL.md` — that is covered by `skill-toolkit/references/rubric.md`.

Background sources: [Claude Code memory docs](https://code.claude.com/docs/en/memory), [Claude Code best practices](https://code.claude.com/docs/en/best-practices), [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice), and community consensus (HumanLayer, Builder.io, vercel, davila7).

Rules are grouped: **A. Structure & length (R01–R06)**, **B. Content quality (R07–R14)**, **C. Mechanism & integration (R15–R19)**, **D. Verifiable heuristics (R20–R22)**.

---

## A. Structure & length

### R01 — Body length budget

Line counts are for the whole file (CLAUDE.md has no frontmatter — see R20). Budgets differ by scope, detected from the file's path:

| Scope                         | Path signal                                             | PASS   | WARN (MEDIUM) | HIGH    | CRITICAL |
|-------------------------------|---------------------------------------------------------|--------|---------------|---------|----------|
| project                       | `./CLAUDE.md`, `./.claude/CLAUDE.md`                    | ≤150   | 151–200       | 201–300 | >300     |
| project (open-source profile) | profile override                                        | ≤200   | 201–250       | 251–350 | >350     |
| user                          | `~/.claude/CLAUDE.md`                                   | ≤30    | 31–60         | 61–100  | >100     |
| local                         | `./CLAUDE.local.md`                                     | ≤60    | 61–100        | 101–150 | >150     |
| managed                       | `/etc/claude-code/CLAUDE.md` and macOS/Windows equivalents | ≤80 | 81–120        | 121–200 | >200     |

Target sweet spot for project scope: 60–150 lines. Anthropic's own guidance says >200 lines causes instruction drop-off.

- **Confidence**: HIGH.

### R02 — Heading hierarchy

- At most ONE `h1`. Many good CLAUDE.md files omit `h1` entirely.
- Main sections use `h2` (`##`), sub-items use `h3` (`###`).
- `h4`+ is forbidden — it dilutes scannability.
- Section titles should be noun phrases, not questions (e.g. `## Testing`, not `## How do I run tests?`).
- **Confidence**: HIGH for structural checks, MEDIUM for noun-phrase heuristic.
- **Severity if violated**: MEDIUM.

### R03 — Critical rules front-loaded

- If the file contains any `IMPORTANT:` / `YOU MUST` / `NEVER` / `MUST NOT` tokens, the section containing them MUST appear before the `Commands` / `Build` / `Testing` sections.
- Rationale: CLAUDE.md is injected as the first user message; early tokens get stronger attention.
- **Confidence**: MEDIUM.
- **Severity if violated**: MEDIUM.

### R04 — Single-topic section size

- No single `h2` section may exceed 30 lines of content (excluding the heading).
- If it does, extract to `@path/to/file.md` import or `.claude/rules/<topic>.md` with `paths:` frontmatter.
- **Confidence**: HIGH.
- **Severity if violated**: MEDIUM.

### R05 — No horizontal-rule decoration

- Flag repeated use of `---` as visual separators in the body. It costs tokens without semantic value.
- Allowed: zero or one `---` total, only if used to delimit a References appendix.
- **Confidence**: HIGH.
- **Severity if violated**: LOW (WARN).

### R06 — Bullet line length

- Bullets SHOULD be ≤1 line each (soft wrap in typical editors). If a bullet runs to 3+ visual lines of prose, convert to a sub-section or move to a linked reference.
- **Confidence**: LOW (heuristic on character count, ~180 chars as threshold).
- **Severity if violated**: LOW (WARN).

---

## B. Content quality

### R07 — Deletability test (verifiability)

- Every instruction should pass the question: "If I delete this line, will Claude make a mistake it wouldn't otherwise make?"
- Heuristic flags: lines that only state facts Claude can read from code, lines that describe the tech stack, lines enumerating obvious file purposes.
- **Confidence**: LOW (semantic judgement).
- **Severity if violated**: MEDIUM, advisory.

### R08 — Concrete over abstract

- Forbidden vague phrases (case-insensitive, in prose only — not in code blocks): `clean code`, `properly`, `best practices`, `good code`, `well-written`, `quality code`, `handle errors gracefully`, `consider performance`, `think about`.
- Rule of thumb: replace each with a machine-checkable assertion (e.g. `function ≤ 50 lines`, `all async functions must try/catch`).
- **Confidence**: HIGH for regex match, MEDIUM for "is the match actually vague in context".
- **Severity if violated**: MEDIUM.

### R09 — Imperative voice

- Instructions SHOULD start with a verb in imperative mood: `Use`, `Run`, `Prefer`, `Avoid`, `Never`.
- Declarative phrasing (`We use 2 spaces`, `Tests should be run`) is weaker for instruction-following.
- **Confidence**: MEDIUM (heuristic on sentence start).
- **Severity if violated**: LOW (WARN).

### R10 — No codebase-duplicative content

Flag any of the following in the body:
- File-by-file descriptions (more than 3 lines of the pattern `` `path/to/file.ext` — description``).
- Tech-stack enumeration with no project-specific addendum (e.g. "We use React, TypeScript, and Node.js" on its own).
- Function signatures copied from code.
- **Confidence**: MEDIUM.
- **Severity if violated**: HIGH.

### R11 — No embedded API docs / tutorials

- Flag continuous prose paragraphs >5 lines. Such content belongs in linked docs (`@docs/api.md`).
- Flag step-by-step tutorials that reiterate what the code self-documents.
- **Confidence**: MEDIUM.
- **Severity if violated**: MEDIUM.

### R12 — No secrets

Regex checks (case-insensitive where applicable):
- OpenAI: `sk-[A-Za-z0-9]{20,}`
- Anthropic: `sk-ant-[A-Za-z0-9-]{20,}`
- GitHub: `ghp_[A-Za-z0-9]{30,}`, `ghs_[A-Za-z0-9]{30,}`, `github_pat_[A-Za-z0-9_]{40,}`
- AWS: `AKIA[0-9A-Z]{16}`, `ASIA[0-9A-Z]{16}`
- Generic bearer: `Bearer\s+[A-Za-z0-9._-]{20,}`
- URL-embedded creds: `://[^\s/]+:[^\s/]+@`
- Private RFC1918 IPs with credentials context
- **Confidence**: HIGH.
- **Severity if violated**: CRITICAL.

### R13 — Emphasis budget

- Total count of `IMPORTANT:` + `YOU MUST` + `MUST` + `NEVER` + `ALWAYS` (standalone, case-sensitive) ≤ 5 per file.
- Rationale: emphasis inflation. If everything is important, nothing is.
- **Confidence**: HIGH.
- **Severity if violated**: MEDIUM.

### R14 — No self-contradiction

Heuristic: detect pairs of rules about the same keyword that give opposing instructions within the same file. Example flags:
- `Use tabs for indentation` and `Use spaces for indentation`
- `Always write tests` and `Skip tests for prototypes`

- **Confidence**: LOW (heuristic, high false-positive rate).
- **Severity if violated**: HIGH when confirmed.

---

## C. Mechanism & integration

### R15 — Advisory-only rules flagged for hook migration

Rules that MUST execute deterministically do not belong in CLAUDE.md (which is advisory, injected as a user message).

Flag any instruction matching: `Always run X after Y`, `Never commit files containing Z`, `Block writes to path P`, where no corresponding entry exists in `.claude/settings.json` (hooks / permissions).

- **Confidence**: LOW (requires cross-file inspection).
- **Severity if violated**: MEDIUM, advisory. Emit the finding with suggested hook / permission config.

### R16 — Import graph sanity

- `@path` imports: maximum 2 hops deep (official allows 5, but practical review limit is 2).
- No cycles.
- Every imported path MUST resolve to a real file (or, for `~/` paths, a path the user's `HOME` can resolve).
- External `@~/` imports require user approval on first encounter — note this in the finding but do not FAIL.
- **Confidence**: HIGH for existence and cycle detection.
- **Severity if violated**: HIGH (broken imports silently drop content).

### R17 — AGENTS.md bridging

If an `AGENTS.md` exists in the same directory:
- CLAUDE.md MUST either (a) contain `@AGENTS.md` as an import, or (b) contain a one-line justification for divergence.
- Content duplication between AGENTS.md and CLAUDE.md (measured by shared line count > 20% of either file) is flagged.
- **Confidence**: HIGH for import-presence check, MEDIUM for duplication.
- **Severity if violated**: HIGH.

### R18 — Scope correctness

- `CLAUDE.local.md` MUST be listed in `.gitignore` (or covered by a broader pattern).
- A project `./CLAUDE.md` MUST NOT contain personal preferences (flag first-person `I` / `my` outside of code blocks) or machine-local paths (`/Users/<name>/`, `/home/<name>/`, `C:\Users\<name>\`).
- A `~/.claude/CLAUDE.md` MUST NOT reference project-specific paths relative to a single repo.
- **Confidence**: HIGH for gitignore and path-leak checks.
- **Severity if violated**: HIGH for gitignore miss; MEDIUM for personal content in shared file.

### R19 — HTML comment usage

- `<!-- ... -->` is stripped before model injection (Claude Code v2.1+), so it is safe for maintainer-only notes.
- Flag any HTML comment that appears to contain instructions intended for Claude (e.g. contains `Use`, `Always`, `Never`, or any IMPORTANT markers) — the model will never see them.
- **Confidence**: MEDIUM.
- **Severity if violated**: MEDIUM.

---

## D. Verifiable heuristics

### R20 — No frontmatter

- CLAUDE.md MUST NOT have YAML frontmatter (no leading `---` block). Frontmatter belongs to `SKILL.md` and `AGENTS.md`, not CLAUDE.md.
- **Confidence**: HIGH.
- **Severity if violated**: HIGH.

### R21 — Commands section resolvability

For each command in a `## Commands` / `## Build` / `## Testing` section (identified by `` `backtick-wrapped` `` code spans that look like shell invocations):

- If a `package.json` exists in the same directory and the command starts with `npm`, `pnpm`, `yarn`, or `bun` followed by a subcommand that maps to a script, verify the script exists.
- If a `Makefile` exists and the command is `make <target>`, verify the target is defined.
- If a `pyproject.toml` exists and the command is `poetry run <x>` or `uv run <x>`, verify the script/task exists.
- Cargo / Gradle / Maven equivalents checked when their config files are present.

Commands that don't match any recognized pattern are skipped (no false positive).

- **Confidence**: HIGH when a config file is present, LOW otherwise.
- **Severity if violated**: HIGH (a broken command command wastes every session).

### R22 — Version-aware features

Flag usage of features that depend on Claude Code v2.1+ without acknowledgement:
- Reliance on HTML comment stripping (R19 behavior) — add a comment like `<!-- requires Claude Code v2.1+ -->` or note in file.
- References to `.claude/rules/` with `paths:` frontmatter — v2.1+.
- References to auto-memory `MEMORY.md` — v2.1.59+.
- Use of `claudeMdExcludes` setting — v2.1+.

- **Confidence**: MEDIUM.
- **Severity if violated**: LOW (WARN). Older Claude Code installs will silently ignore these features.

---

## Confidence summary

| Confidence | Rules |
|------------|-------|
| HIGH       | R01, R02 (structural), R04, R05, R08 (regex), R12, R13, R16, R17 (import), R18 (gitignore, paths), R20, R21 (when config present) |
| MEDIUM     | R02 (noun-phrase), R03, R08 (context), R09, R10, R11, R17 (duplication), R18 (personal content), R19, R22 |
| LOW        | R06, R07, R14, R15, R21 (unrecognized commands) |

## Severity summary

| Severity | Typical triggers |
|----------|------------------|
| CRITICAL | R01 worst-tier, R12 |
| HIGH     | R01 high-tier, R10, R14 (confirmed), R16, R17, R18 (gitignore), R20, R21 |
| MEDIUM   | R02, R03, R04, R07, R08, R11, R13, R15, R18 (personal), R19 |
| LOW      | R05, R06, R09, R22 |

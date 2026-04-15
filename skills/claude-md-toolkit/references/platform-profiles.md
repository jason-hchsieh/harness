# Platform profiles

Profile-specific adjustments to the rubric and skeleton. A profile tunes defaults without changing rule semantics — the rubric is still R01–R22, but thresholds and required sections shift.

Used by `mode-create.md` (scaffold selection) and `mode-audit.md` (R01 budget override, required-section checks).

## Profile selection algorithm

Detect by scanning the target's directory for signals, in order. First match wins:

1. `pyproject.toml` / `requirements.txt` referencing `torch`, `tensorflow`, `transformers`, `wandb`, `mlflow`, `jax`, `pytorch-lightning`; OR the presence of `models/`, `datasets/`, `experiments/`, `notebooks/` at repo root → **ml**
2. `package.json` with `workspaces` field, `pnpm-workspace.yaml`, `turbo.json`, `lerna.json`, `nx.json`, `rush.json` → **monorepo**
3. `SECURITY.md` containing HIPAA / SOC2 / PCI-DSS / GDPR / ISO27001 keywords, OR a `compliance/`, `audit/`, `soc2/` directory → **compliance**
4. `CONTRIBUTING.md` + `LICENSE` + a README with install badges / public-facing tone → **open-source**
5. Otherwise → **default**

Confirm the detected profile with the user before applying in `create` mode. In `audit` mode, apply silently but emit the detected profile in the report header so the user can correct.

## Profile: default

- **R01 budget**: project ≤150 (PASS), 151–200 (WARN), 201–300 (HIGH), >300 (CRITICAL).
- **Required sections**: Overview, Commands.
- **Recommended sections**: Project Structure, Code Style, Testing, Gotchas.
- No additional rules.

## Profile: monorepo

**Intent**: root CLAUDE.md stays small; per-package CLAUDE.md carries the detail.

- **R01 budget**: root project ≤100 (PASS), 101–150 (WARN), 151–250 (HIGH), >250 (CRITICAL).
- **Required sections at root**: Overview, Repo Layout (list packages), Shared Commands.
- **Forbidden at root**: package-specific commands or architecture. Those belong in `packages/<name>/CLAUDE.md`.
- **Additional checks**:
  - Audit emits MEDIUM if root CLAUDE.md contains a command that only works inside one package.
  - Audit checks `.claude/settings.json` for `claudeMdExcludes` if root has >3 sibling packages; suggests enabling it to exclude unrelated `CLAUDE.md` from load.
  - Each `packages/*/CLAUDE.md` is audited independently (with `project` scope, `default` profile unless it has its own signals).

## Profile: ml

**Intent**: extra sections for ML-specific concerns that other profiles don't need.

- **R01 budget**: project ≤180 (PASS), 181–230 (WARN), 231–330 (HIGH), >330 (CRITICAL). Slightly higher because ML projects genuinely need more context.
- **Required sections**: Overview, Commands, Data / Datasets, Experiment Tracking, Checkpoint Conventions.
- **Recommended sections**: Model Architecture Notes, GPU/CPU Switching, Long-Run Warnings.
- **Additional checks**:
  - Audit emits HIGH if the `## Data` section references dataset paths without noting size, location, or access requirements (e.g. "S3 bucket requires AWS SSO").
  - Audit emits MEDIUM if training commands appear without a cost/runtime warning (e.g. "full training run ≈18h on 8×A100").
  - Audit emits MEDIUM if checkpoint naming is not specified (leads to overwrite accidents in `Edit`-happy agents).
- **Anti-pattern specific to ML**: "Try different hyperparameters" — R08 vague. Replace with concrete starting points or a sweep config reference.

## Profile: open-source

**Intent**: CLAUDE.md doubles as contributor onboarding, so it may be longer and more narrative.

- **R01 budget**: project ≤200 (PASS), 201–250 (WARN), 251–350 (HIGH), >350 (CRITICAL).
- **Required sections**: Overview, Quick Start, Commands, Contribution Workflow.
- **Recommended sections**: Project Structure, Testing, Code Style, How to File an Issue.
- **Additional checks**:
  - Audit emits HIGH if internal URLs (`*.internal.example.com`, `*.corp.*`, `localhost:*` with org-specific ports) appear.
  - Audit emits HIGH if Jira / Linear ticket patterns (`[A-Z]{2,}-\d+`) appear — use GitHub issue numbers instead.
  - Audit emits MEDIUM if Slack / Discord channel names appear without noting they are internal.
  - Audit emits MEDIUM if the file assumes the reader has internal tooling installed.
- **Security note**: open-source CLAUDE.md is a prompt-injection surface. A malicious contributor can PR content that overrides reviewer intent. Mitigations (out of scope for this toolkit but worth noting):
  - PR review should flag CLAUDE.md changes.
  - Consider `CODEOWNERS` entries for CLAUDE.md.

## Profile: compliance

**Intent**: files in regulated environments (healthcare, finance, payments). Emphasis on secret hygiene, audit trail, and deterministic enforcement.

- **R01 budget**: project ≤150 (PASS), 151–200 (WARN), 201–300 (HIGH), >300 (CRITICAL). Same as default — compliance discipline does NOT justify bloat.
- **Required sections**: Overview, Critical Rules, Data Handling, Commands.
- **Recommended sections**: Audit Trail, Access Control, Incident Response pointers.
- **Additional checks**:
  - R12 (secrets) regex is extended: add HIPAA/PHI keywords (`SSN`, `MRN`, `DOB`), PCI patterns (`\d{13,19}` digit clumps adjacent to "card"/"pan"), IBAN-looking strings.
  - Audit emits HIGH if any instruction relies on model compliance alone for regulated behavior — R15 is upgraded from MEDIUM to HIGH. Compliance rules MUST be enforced by hooks / permissions / sandbox.
  - Audit emits HIGH if PHI/PII example data appears in the file (even synthetic-looking — no real-looking data).
  - Audit emits MEDIUM if Managed scope is not mentioned ("see organization-wide policy in managed CLAUDE.md").
- **Required cross-reference**: compliance CLAUDE.md should mention the associated managed policy or organizational doc.
- **Forbidden in compliance CLAUDE.md**:
  - Real or realistic customer identifiers.
  - Internal system names that could aid an attacker (e.g. production DB hostnames).
  - Credential examples, even fake-looking ones.

## Profile override

Users can force a profile in `create` mode by passing it as the second argument:
`claude-md-toolkit create project --profile=ml`

Audit does not accept a profile override in v0.1.0 — detection is automatic. (Future: allow `--profile=X` flag for audit to test "what would this look like under profile Y".)

## Interaction with scope

Scope (project / user / local / managed) and profile (default / monorepo / ml / open-source / compliance) are orthogonal:

- Scope determines the target path and the git/gitignore behavior.
- Profile determines the rubric thresholds and required sections.

Profile only applies to project-scope files in practice. User / local / managed scope files always use the default profile.

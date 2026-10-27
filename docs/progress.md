# Progress

## Current state
- Workflow template pushed to `main`. The repo is marked as a template.
- #1 (`.gitattributes` LF, `ready` rules to `ask`) merged via PR #2.
- Issue #3 is in progress on branch `chore/3-action-label-hard-block`. It hard-blocks
  unattended `ready` label changes in `ready.yml` via `--disallowedTools`. PR not opened yet.

## Open issues / PRs
- #3 (chore): hard-block `ready` label changes for unattended runs in `ready.yml`
  (`claude_args --disallowedTools`). Branch `chore/3-action-label-hard-block`, PR pending
  (will close #3). Reviewer: APPROVE after 2 rounds, no blockers. Known gaps are in
  `docs/decisions.md` (2026-10-27).

## Next
- [ ] **Open the PR** for `chore/3-action-label-hard-block` with `Closes #3`.
- [ ] **First task once the stack is chosen:** the conductor must fill in
  `.github/workflows/ci.yml` (install, test and lint steps) and the commands in
  root `CLAUDE.md`. `ci.yml` is a placeholder until then.

## Done
- #1 `.gitattributes` for LF line endings and `ready` rules moved from `deny` to `ask`
  (PR #2, merged).

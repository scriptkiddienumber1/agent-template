# Progress

## Current state
- Workflow template pushed to `main`. The repo is marked as a template.
- #1 (`.gitattributes` LF, `ready` rules to `ask`) merged via PR #2.
- #3 (hard-block unattended `ready` label changes, start-of-issue sync) merged via PR #4
  (`Closes #3`).
- Issue #5 is in progress on branch `chore/5-unattended-dirty-tree-label-guard`. PR not opened
  yet; it will say `Closes #5`. Reviewer: APPROVE after 2 rounds. Scope:
  - Unattended dirty tree: the conductor does not wait. It comments on the issue, adds
    `needs-decision` and stops the run. Interactive behavior is unchanged.
  - `ready.yml` `guard` job: when anyone but the repository owner adds `ready`, it removes the
    label (REST DELETE, no Claude) and comments why. The `conductor` job runs only for the owner.
    The `ready-queue` concurrency group moved to the `conductor` job.
  - Not tested in a real Action run.

## Open issues / PRs
- #5 (chore, branch `chore/5-unattended-dirty-tree-label-guard`): in progress. PR pending.
  Known nits (see `docs/decisions.md`, 2026-10-27): a tiny race with a conductor run that already
  listed `ready` issues; a 404 gives a harmless false-alarm comment.
- Optional follow-ups from PR #4, skipped by the owner (not applied; see `docs/decisions.md`,
  2026-10-27):
  - Set upstream explicitly: `git switch -c <branch> origin/main` tracks `origin/main`
    (use `--no-track`, or `git push -u origin <branch>`).
  - Name the existing-branch check commands (`git branch --list`,
    `git branch -r --list origin/<branch>`) and add `git branch` to the read-only git list.

## Next
- [ ] **Open the PR for #5** (`Closes #5`, what changed and why, how to test). Never merge;
  merging is the user's decision.
- [ ] **Test #5 in a real Action run** (untested): a non-owner `ready` is removed with a comment;
  an owner `ready` starts the conductor; a dirty tree in an unattended run comments and stops.
- [ ] **Before using the template with an org-owned repo:** replace `github.repository_owner` in
  both `if` expressions of `.github/workflows/ready.yml` with the allowed login.
- [ ] **Optional:** apply the two follow-ups above to `conductor.md` (owner approval needed).
- [ ] **First task once the stack is chosen:** the conductor must fill in
  `.github/workflows/ci.yml` (install, test and lint steps) and the commands in
  root `CLAUDE.md`. `ci.yml` is a placeholder until then.

## Done
- #1 `.gitattributes` for LF line endings and `ready` rules moved from `deny` to `ask`
  (PR #2, merged).
- #3 hard-block of unattended `ready` label changes and the start-of-issue sync
  (PR #4, merged).

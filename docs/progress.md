# Progress

## Current state
- Workflow template pushed to `main`. The repo is marked as a template.
- #1 (`.gitattributes` LF, `ready` rules to `ask`) merged via PR #2.
- #3 (hard-block unattended `ready` label changes, start-of-issue sync) merged via PR #4
  (`Closes #3`).
- #5 (unattended dirty tree stop, owner-only `ready` guard job) merged via PR #6 (`Closes #5`).
  Not yet tested in a real Action run.
- Short final summaries (branch `chore/short-final-summaries`, no issue) merged via PR #7.
- #8 (token efficiency: 5-line subagent reports, size-scaled review, explicit models, unattended
  `--max-turns` 80) merged via PR #9 (`Closes #8`). Untested in a real run.
- Trivial-change tie-break (branch `chore/trivial-tie-break`, commit 44ffb1a, no issue) adds "when
  in doubt, treat the change as non-trivial" to `conductor.md`. Owner-requested. Reviewer verdict
  APPROVE, no findings. PR pending (number to add once opened).

## Open issues / PRs
- Pending PR: `chore/trivial-tie-break` (no issue). Decisions in `docs/decisions.md`, 2026-10-27.
  Next step: push the branch and open the PR. Never merge; merging is the user's decision.
- #5 is merged (PR #6). Known nits remain (see `docs/decisions.md`, 2026-10-27): a tiny race with
  a conductor run that already listed `ready` issues; a 404 gives a harmless false-alarm comment.
- Skipped by owner (not applied; see `docs/decisions.md`, 2026-10-27):
  - The "Subagent reports" section in `conductor.md` repeats the briefing rule from step 4.
- Optional follow-ups from PR #4, skipped by the owner (not applied; see `docs/decisions.md`,
  2026-10-27):
  - Set upstream explicitly: `git switch -c <branch> origin/main` tracks `origin/main`
    (use `--no-track`, or `git push -u origin <branch>`).
  - Name the existing-branch check commands (`git branch --list`,
    `git branch -r --list origin/<branch>`) and add `git branch` to the read-only git list.

## Next
- [ ] **Push `chore/trivial-tie-break` and open its PR** (no issue, so no `Closes`). Never merge;
  merging is the user's decision.
- [ ] **Test #5 in a real Action run** (untested): a non-owner `ready` is removed with a comment;
  an owner `ready` starts the conductor; a dirty tree in an unattended run comments and stops.
- [ ] **Test #8 in a real run** (untested): 5-line reports, trivial changes get one review round,
  models apply, and the 80-turn cap is enough for a normal issue.
- [ ] **Before using the template with an org-owned repo:** replace `github.repository_owner` in
  both `if` expressions of `.github/workflows/ready.yml` with the allowed login.
- [ ] **Optional:** apply the remaining optional follow-ups above to `conductor.md` (owner approval
  needed).
- [ ] **First task once the stack is chosen:** the conductor must fill in
  `.github/workflows/ci.yml` (install, test and lint steps) and the commands in
  root `CLAUDE.md`. `ci.yml` is a placeholder until then.

## Done
- #1 `.gitattributes` for LF line endings and `ready` rules moved from `deny` to `ask`
  (PR #2, merged).
- #3 hard-block of unattended `ready` label changes and the start-of-issue sync
  (PR #4, merged).
- #5 unattended dirty-tree stop and owner-only `ready` guard job (PR #6, merged).
- Short final summaries (PR #7, merged, no issue).
- #8 token efficiency (PR #9, merged).

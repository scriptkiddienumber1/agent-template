# Progress

## Current state
- Workflow template pushed to `main`. The repo is marked as a template.
- #1 (`.gitattributes` LF, `ready` rules to `ask`) merged via PR #2.
- Issue #3 is in PR #4 (branch `chore/3-action-label-hard-block`, `Closes #3`). Scope:
  - `ready.yml` hard-blocks unattended `ready` label changes via `claude_args --disallowedTools`,
    and also passes the server-level deny `mcp__github` (defense in depth, see decisions).
  - `conductor.md` section 3 has the start-of-issue sync: continue on a checked-out issue
    branch; otherwise a clean tree gets `git fetch origin`, then switch to an existing branch
    or create the new one from the latest `origin/main`. A dirty tree stops work.
    `.claude/settings.json` allow list gained `Bash(git fetch *)` and `Bash(git pull --ff-only *)`.

## Open issues / PRs
- #3 (chore): hard-block `ready` label changes for unattended runs, plus the start-of-issue
  sync. Closed by PR #4 when merged. Known gaps and the MCP check are in `docs/decisions.md`
  (2026-10-27).
- PR #4 (chore, branch `chore/3-action-label-hard-block`): open, `Closes #3`. Reviewer:
  APPROVE after 2 rounds for the scope extension. Not merged; merging is the user's decision.
- Optional follow-ups for the owner (not blocking, not applied; see decisions 2026-10-27):
  - Unattended runs: a dirty tree currently means "STOP and tell the user", which would halt
    the queue silently. Suggested: when unattended, record it in `docs/progress.md`, comment on
    the issue and stop that issue.
  - Set upstream explicitly: `git switch -c <branch> origin/main` tracks `origin/main`
    (use `--no-track`, or `git push -u origin <branch>`).
  - Name the existing-branch check commands (`git branch --list`,
    `git branch -r --list origin/<branch>`) and add `git branch` to the read-only git list.

## Next
- [ ] **Review and merge PR #4** (user's decision). Then close out #3.
- [ ] **Optional:** apply the follow-ups above to `conductor.md` (owner approval needed).
- [ ] **First task once the stack is chosen:** the conductor must fill in
  `.github/workflows/ci.yml` (install, test and lint steps) and the commands in
  root `CLAUDE.md`. `ci.yml` is a placeholder until then.

## Done
- #1 `.gitattributes` for LF line endings and `ready` rules moved from `deny` to `ask`
  (PR #2, merged).
</content>

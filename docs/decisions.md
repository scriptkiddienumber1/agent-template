# Decisions

Newest first. Format: date, decision, why, alternatives considered.

## 2026-10-27

- **Start every issue from the latest `origin/main`** (conductor flow, `.claude/agents/conductor.md`
  section 3; issue #3 scope extension).
  Decision: if the issue branch is already checked out, continue on it without switching.
  Otherwise, on a clean tree, run `git fetch origin`, then: an existing branch is switched to
  (never recreated); a new branch is made with `git switch main`, `git pull --ff-only origin main`,
  `git switch -c <branch> origin/main`. A dirty tree stops work and nothing is switched.
  Why: branches start from current `main` (no stale bases, fewer conflicts), and uncommitted
  local work is never clobbered.
  Permissions: the conductor's Bash exception names `git fetch`, `git switch` and
  `git pull --ff-only origin main`. The `.claude/settings.json` allow list gained
  `Bash(git fetch *)` and `Bash(git pull --ff-only *)`.
  Alternatives: not recorded.
  Known follow-ups (not applied; owner's call):
  - Unattended runs: "STOP and tell the user" after a dirty tree would halt the whole queue
    silently. Suggested: when unattended, record it in `docs/progress.md`, comment on the issue
    and stop that issue.
  - `git switch -c <branch> origin/main` sets `origin/main` as upstream. Use `--no-track`, or
    push with `git push -u origin <branch>`.
  - The existing-branch check names no command. Suggested: `git branch --list` and
    `git branch -r --list origin/<branch>`, and add `git branch` to the read-only git list in
    the conductor intro.

- **Hard-block unattended `ready` label changes in `ready.yml`** (issue #3). `claude_args`
  now also passes `--disallowedTools` with `Bash(gh issue edit *ready*)`,
  `Bash(gh issue create *ready*)`, `Bash(gh * issue edit *ready*)`,
  `Bash(gh * issue create *ready*)` (catches `gh -R owner/repo issue ...`),
  `Bash(gh api *)`, `Bash(gh pr edit *)` and the server-level `mcp__github`.
  Why: the only protection before was an `ask` rule in the shared `.claude/settings.json`.
  Unattended runs cannot answer prompts, but if that file were loosened, runs could add
  `ready`. This rule lives in the Action's own args, so it holds even if the settings change.
  Mechanism: the old `disallowed_tools` input is deprecated; claude-code-action documents
  `claude_args: --disallowedTools ...` as the replacement.
  Trade-offs and gaps:
  - Best-effort. It matches `gh` command strings only. curl or Python using the token,
    obfuscated commands or aliases could still reach the API. A hard guarantee would need
    another mechanism (e.g. a step that reverts the label or checks the label actor).
    Backstop: labels added with `GITHUB_TOKEN` do not trigger new workflow runs.
  - `*ready*` is a substring match, so unrelated commands containing "ready" are refused too.
    It fails closed.
  - `gh pr edit *` and `gh api *` are blocked for unattended runs entirely. The flow creates
    PRs with `gh pr create` and does not need them.
  - MCP check: the Action's own MCP tools (`github_comment`, `github_inline_comment`,
    `github_file_ops`, `github_ci`) have no label tool. The official GitHub MCP server
    (`mcp__github__*`, v0.17.1 image; `update_issue`/`create_issue` take labels, newer versions
    `issue_write`) appears, from the Action's source, to be registered only when `allowedTools`
    names it. `ready.yml` does not. `--disallowedTools "mcp__github"` (server-level deny) is
    defense in depth on top of that.
  - Unverified: that `--disallowedTools` applies to MCP names inside the Action. `base-action/`
    was not checked.
  Alternative: pass the rules through the Action's `settings` input. Not chosen, because
  `claude_args --disallowedTools` is the documented replacement.

- **Use `ask` instead of `deny` for `ready` label commands.** The rules
  `Bash(gh issue edit *ready*)` and `Bash(gh issue create *ready*)` moved from
  `permissions.deny` to `permissions.ask` in `.claude/settings.json`.
  Why: the owner adds `ready` after a yes in chat by approving a prompt. `deny` blocked
  even that owner-approved case. Unattended runs still cannot add `ready`, because
  nobody can approve the prompt and the command is refused.
  Trade-off: the `*ready*` pattern also prompts for any `gh issue edit`/`create` whose
  text contains "ready" (e.g. in a title).
  Alternatives: keep `deny` (blocks the owner too).

- **Normalize line endings to LF** with root `.gitattributes` (`* text=auto eol=lf`).
  Why: the repo is edited on Windows, while CI and Action runners are Linux.
  Alternatives: not recorded.

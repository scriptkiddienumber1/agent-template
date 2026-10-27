# Decisions

Newest first. Format: date, decision, why, alternatives considered.

## 2026-10-27

- **Trivial-change tie-break in `conductor.md`.** Added "when in doubt, treat the change as
  non-trivial". Why: avoid under-reviewing a change that only looks small. Skipped nit: the
  duplicate briefing rule in the "Subagent reports" section, skipped by the owner.

- **Token efficiency: short subagent reports, size-scaled review, explicit models, lower turn cap**
  (issue #8; `.claude/agents/conductor.md` steps 3 and 6-7 and "Subagent reports"; agent model
  lines; `.github/workflows/ready.yml` `--max-turns`).
  - Subagents (builder, reviewer, docs) report to the conductor in max 5 lines. Details go in
    commits and the PR description.
  - Conductor briefs subagents with the issue number, relevant files and goal only. No conversation
    history.
  - Size scaling: trivial changes (config, docs, one-liners) get max 1 review round and run `docs`
    only if project state or a decision changed. Other changes keep max 2 rounds. Review is never
    skipped.
  - Models: builder and reviewer `model: sonnet`, docs `model: haiku`. Conductor model is unset and
    inherits the session's. Per project, the builder can be raised (e.g. to opus) if sonnet is not
    good enough for the codebase. Record that choice here.
  - Unattended `--max-turns` lowered from 200 to 80. This bounds a stuck unattended run.
  Why: long reports and full cycles on one-liners cost tokens. History-heavy briefs are expensive
  and not needed. Implementation and review work fits cheaper models.
  Alternatives: keep the full process always (rejected: cost); keep all agents on the session model
  (rejected: cost).
  Status: reviewer APPROVE, no blockers. Untested in a real run.
  Open nits (optional, not applied):
  - "Trivial" has no tie-break ("when in doubt, treat as non-trivial"). It also does not cover a
    config change that alters behavior or the shared contract.
  - The "Subagent reports" section repeats the briefing rule from step 4.

- **Short final summaries to the owner** (`.claude/agents/conductor.md` step 9; root `CLAUDE.md`
  GitHub workflow). Format: Done (one line), Needs me (one line per item, omitted if none),
  Caveats (one line per real risk or untested assumption, omitted if none), PR link. Details go in
  the PR description. Chat detail only if something went wrong or the owner asks.
  Why: the owner finds long summaries costly to read. The PR is the durable place for detail, and
  files are the memory.
  Alternative: keep long summaries in chat. Rejected for the reason above.
  Made without an issue at the owner's request, so branch `chore/short-final-summaries` has no
  issue number.

- **Unattended dirty tree: comment, add `needs-decision`, stop the run**
  (`.claude/agents/conductor.md` section 3 step 2; `ready.yml` prompt). Applies to unattended runs
  only. The conductor does not wait: it comments on the issue what is dirty, adds `needs-decision`
  and stops the run. Interactive runs are unchanged: they stop and tell the user.
  Why: nobody can answer in an unattended run, and the next issue could not start on the dirty
  tree either. Before this, the run halted silently.
  Alternative: skip that issue and continue. Rejected for the reason above.

- **Owner-only `ready`: `guard` job in `ready.yml`.** A new `guard` job runs when `ready` is added
  by anyone other than `github.repository_owner`. It removes the label with
  `gh api -X DELETE repos/$REPO/issues/$ISSUE/labels/ready` and comments why. If the removal fails,
  it still comments and exits non-zero (red run). The `conductor` job runs only when the sender is
  the owner, so it never starts before the guard acts. No Claude is involved.
  Why: the `claude_args` block is best-effort (it matches `gh` command strings), so it cannot
  enforce owner-only `ready` alone. The guard enforces it in the workflow itself.
  Concurrency: the `ready-queue` group moved from workflow level to the `conductor` job, so one
  conductor runs at a time and the rest queue. The guard has its own group per issue
  (`ready-guard-<n>`), so it never waits behind a long conductor run.
  Permissions: the REST DELETE needs only `issues: write`, which keeps `GITHUB_TOKEN` at minimum.
  Allowed user: `github.repository_owner`, not a hardcoded login, so repos created from the
  template keep working. Caveat: for an organisation-owned repo this is the org name, which no
  sender matches. The guard would strip every `ready` and the conductor would never run. Before
  using the template with an org-owned repo, replace `github.repository_owner` in both `if`
  expressions with the allowed login. `env` is not available in job-level `if`, which is why the
  context is used directly.
  Alternatives: (1) rely only on the `claude_args` block (rejected: best-effort). (2) `gh issue edit`
  instead of REST (rejected for `GITHUB_TOKEN` permission safety). (3) a hardcoded login (rejected:
  breaks the template).
  Known nits (accepted):
  - Tiny race: a conductor run that already listed `ready` issues could pick one up before the
    guard removes the label.
  - If the label was already removed by someone else, the DELETE returns 404. The guard then posts
    a harmless false-alarm comment.
  - Untested in a real Action run.

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
  Known follow-ups (owner's call):
  - Unattended runs: applied in the 2026-10-27 unattended dirty-tree entry above (the original
    suggestion was to stop only that issue; the run is now stopped, since the next issue could not
    start either).
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

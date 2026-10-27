# agent-template

A reusable template for an agent-driven workflow with Claude Code: a conductor
plans and delegates to builder, reviewer and docs agents; GitHub issues are the
backlog; you decide what is `ready` and what gets merged.

## Start a project from this template
1. Click **Use this template** on GitHub (or clone and re-init), then clone locally.
2. Run `claude` in the repo. `.claude/settings.json` starts the **conductor**.
3. Because `CLAUDE.md` says `Project: TBD`, the conductor interviews you, then has
   `docs` write `CLAUDE.md` and `docs/decisions.md` before any building starts.
4. Refine issues with the conductor. Say yes to `ready` when an issue should be
   started right away.
5. First task once the stack is chosen: fill in `.github/workflows/ci.yml`.

## One-time setup (per repo)
- Install the [Claude GitHub App](https://github.com/apps/claude) on the repo (or run `/install-github-app`).
- Run `claude setup-token` locally and add the result as the repo secret
  `CLAUDE_CODE_OAUTH_TOKEN` (uses your Claude subscription). Alternative: an API
  key as `ANTHROPIC_API_KEY`, swapping the auth line in `ready.yml`.
- Install and authenticate the GitHub CLI: `gh auth login`.
- Create labels: `gh label create bug`, `feature`, `chore`, `ready`, `needs-decision`
  (the conductor does this if missing).
- Branch protection on `main`: require PRs and the CI check; no direct pushes.
- Optional routines: see `docs/routines.md` and set them up with `/schedule`.

## Layout
- `.claude/agents/` – conductor, builder, reviewer, docs
- `.claude/settings.json` – conductor as main agent, permission rules
- `.github/workflows/ready.yml` – runs the conductor when an issue gets `ready`
- `.github/workflows/ci.yml` – placeholder tests + lint on PRs
- `docs/` – `progress.md`, `decisions.md`, `routines.md`

## Notes
- Each `ready.yml` run works through *all* open `ready` issues without an open PR,
  oldest first, one at a time, until none remain or the budget (`--max-turns 200`,
  120-minute timeout) is hit. Runs queue one at a time; GitHub keeps only the newest
  pending run, which is fine because every run drains the whole backlog. Issues left
  over are picked up by the next run.
- `needs-decision` (`gh issue edit ... --add-label needs-decision`) is the one label
  unattended runs may add, with no prompt. Commands mentioning "ready" require approval
  (ask rule). The owner can approve adding `ready` interactively after saying yes in
  chat. Unattended runs are refused because nobody can approve, so agents never add
  `ready` on their own. Other `gh issue edit` forms also need approval.
- Unattended runs are also hard-blocked in `ready.yml` via `--disallowedTools`
  (best-effort: matches `gh` command strings, so other tools could still reach the API).
  See `docs/decisions.md`.

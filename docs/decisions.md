# Decisions

Newest first. Format: date, decision, why, alternatives considered.

## 2026-10-27

- **Hard-block unattended `ready` label changes in `ready.yml`** (issue #3). `claude_args`
  now also passes `--disallowedTools` with `Bash(gh issue edit *ready*)`,
  `Bash(gh issue create *ready*)`, `Bash(gh * issue edit *ready*)`,
  `Bash(gh * issue create *ready*)` (catches `gh -R owner/repo issue ...`),
  `Bash(gh api *)` and `Bash(gh pr edit *)`.
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
  - The Action's built-in MCP tools (comment, inline comment, file ops, CI) cannot add
    labels. The official `mcp__github__*` server was not checked.
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

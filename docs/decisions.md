# Decisions

Newest first. Format: date, decision, why, alternatives considered.

## 2026-10-27

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

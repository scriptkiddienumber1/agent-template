# Project: TBD

Run the conductor (default agent in `.claude/settings.json`); it will interview
you and fill this file in.

## Principles
- Files are the memory, not the chat. Record state in `docs/progress.md`, choices in `docs/decisions.md`.
- Anything both sides of the codebase need goes in this file. Anything only one
  area needs goes in that folder's own `CLAUDE.md`.
- Keep this file short.

## Shared contract
_TBD: API/interface between components. Changing it requires the user's approval._

## Commands
_TBD: install, test, lint, run._

## GitHub workflow (gh CLI)
- Issues are the backlog. Labels: `bug`, `feature`, `chore`, `ready`, `needs-decision`.
  Create any that are missing with `gh label create`.
- One branch per issue: `<type>/<issue-number>-<short-slug>` (type = bug | feature | chore).
  Every issue branch is created from the latest `origin/main` after `git fetch` and
  `git pull --ff-only origin main`. If the working tree is dirty, stop and tell the user.
- PRs include `Closes #<n>`, what changed and why, and how to test.
- Never merge PRs. Never push to `main`. Merging is always the user's decision.
- `ready` = start as soon as possible. Only the user approves it; agents and
  routines never add it on their own.
- Anything needing the user's input: add `needs-decision` with a comment
  explaining the question, then stop work on that issue.
- Interruption safety: builder commits and pushes the issue branch after each
  meaningful step (WIP is fine). Conductor comments "in progress" when starting an
  issue and records exactly where it stopped and the next step in
  `docs/progress.md` before stopping. On resume, read `docs/progress.md`, open
  issues and branches without a PR, and continue rather than restart.
- Every behavior change ships with tests and goes through the reviewer first.

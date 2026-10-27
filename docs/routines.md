# Routines

Prompts for scheduled routines. **Not active in the template.** Set each up per
project with `/schedule`, pasting the prompt below.

Rules for every routine: never merge, never deploy, never add the `ready` label,
never work on an issue that does not already have `ready`, never push to `main`.

## Status digest
```
Summarize the state of this repo since the last digest:
- PRs awaiting review (number, title, age, CI status)
- Issues labeled needs-decision, with the open question from the latest comment
- Progress since the last digest (merged PRs, new issues, changes in docs/progress.md)
Read-only: post nothing except the digest itself (as a comment on a pinned
"Digest" issue, or in the run output). Do not change labels or code.
```

## Fallback backlog pickup
Only needed if `.github/workflows/ready.yml` cannot run (e.g. Actions down or the token expired). `ready.yml` already drains the whole backlog per run.
```
Find the oldest open issue labeled "ready" that has no open PR (check
`gh pr list` for "Closes #<n>" / branch names). If there is none, exit
immediately. Otherwise work on that single issue only, following the conductor
flow in CLAUDE.md (builder -> reviewer, max 2 rounds -> docs -> open PR).
One issue per run. If input is needed, comment the question, request the
needs-decision label, and stop. Never merge, never add "ready".
```

## Weekly health check
```
Run the test suite and lint. Check dependencies for outdated and vulnerable
packages using the project's package manager audit tools. For each distinct
problem, open a GitHub issue (label chore or bug) with details and reproduction.
Skip problems that already have an open issue. Never fix anything directly, never
open PRs, never add "ready".
```

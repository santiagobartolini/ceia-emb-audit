# CLAUDE.md

## Agent skills

### Issue tracker

Issues live as GitHub issues in `santiagobartolini/ceia-emb-audit`, using the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default label vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout: `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Git

Never add "Co-Authored-By: Claude", "Generated with Claude Code", or any claude.ai session link to commit messages or PR descriptions.

Never open pull requests. Only commit and push to the current session branch; pull requests are created automatically by CI.

Use Conventional Commits for every commit message (feat:, fix:, docs:, chore:, refactor:, test:). Write the entire message in English, both the subject line and the body. Keep the subject short and in the imperative mood.

## Code style

Write all identifiers, docstrings and code comments in English, including test names.

Write user-facing text in Spanish: web UI copy, audit report output, and CLI messages aimed at teachers or students.

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

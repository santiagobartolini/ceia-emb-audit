# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues.

Use `gh api` with GitHub's REST API for every operation. Claude Code cloud sessions block GitHub's GraphQL API, and most `gh issue`, `gh pr` and `gh label` subcommands use it, so they fail there with HTTP 403: never use them. In `gh api` paths, `{owner}` and `{repo}` are filled in automatically from the clone's git remote. With `-f` or `-F` fields, `gh api` sends a POST unless `-X` says otherwise.

## Conventions

- **Create an issue**: `gh api repos/{owner}/{repo}/issues -f title="..." -F body=@- --jq .number`, passing the body on stdin through a heredoc. Add labels with `-f 'labels[]=<label>'`, once per label.
- **Read an issue**: `gh api repos/{owner}/{repo}/issues/<number> --jq '{number, title, state, body, labels: [.labels[].name], assignees: [.assignees[].login]}'`, then its comments with `gh api repos/{owner}/{repo}/issues/<number>/comments --paginate --jq '.[] | {author: .user.login, body}'`.
- **List issues**: `gh api 'repos/{owner}/{repo}/issues?state=open&per_page=100' --paginate --jq '.[] | select(.pull_request | not) | {number, title, labels: [.labels[].name]}'`. Filter by adding `&labels=<label>` or changing `state` in the query string. The REST list also returns pull requests, which the `select` drops.
- **Comment on an issue**: `gh api repos/{owner}/{repo}/issues/<number>/comments -f body="..."` for one line, or `-F body=@-` with a heredoc for multi-line bodies.
- **Apply / remove labels**: `gh api repos/{owner}/{repo}/issues/<number>/labels -f 'labels[]=<label>'` / `gh api -X DELETE repos/{owner}/{repo}/issues/<number>/labels/<label>`.
- **Close**: comment first if there is something to say, then `gh api -X PATCH repos/{owner}/{repo}/issues/<number> -f state=closed`.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the REST equivalents:

- **Read a PR**: `gh api repos/{owner}/{repo}/pulls/<number>`, its comments with `gh api repos/{owner}/{repo}/issues/<number>/comments`, and the diff with `gh api repos/{owner}/{repo}/pulls/<number> -H 'Accept: application/vnd.github.diff'`.
- **List external PRs for triage**: `gh api 'repos/{owner}/{repo}/pulls?state=open&per_page=100' --paginate --jq '.[] | {number, title, body, labels: [.labels[].name], author: .user.login, author_association}'`, then keep only `author_association` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / label / close**: comments and labels use the issue endpoints above with the PR number; close with `gh api -X PATCH repos/{owner}/{repo}/pulls/<number> -f state=closed`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either: `gh api repos/{owner}/{repo}/issues/42` returns both, and a PR carries a `pull_request` key.

## When a skill says "publish to the issue tracker"

Create a GitHub issue, as described above.

## When a skill says "fetch the relevant ticket"

Read the issue and its comments, as described above.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. Create it as above with `-f 'labels[]=wayfinder:map'`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue: `gh api repos/{owner}/{repo}/issues/<map>/sub_issues -F sub_issue_id=<child-db-id>`, where `<child-db-id>` is a database id as described under Blocking. Where sub-issues aren't enabled, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies**, the canonical, UI-visible representation. Add an edge with `gh api --method POST repos/{owner}/{repo}/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, where `<blocker-db-id>` is the blocker's numeric **database id** (`gh api repos/{owner}/{repo}/issues/<n> --jq .id`, _not_ the `#number` or `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only, the live gate). Where dependencies aren't available, fall back to a `Blocked by: #<n>, #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children (`gh api repos/{owner}/{repo}/issues/<map>/sub_issues --paginate`, keeping `state` open, or the map's task list), drop any with an open blocker (`issue_dependencies_summary.blocked_by > 0`, or an open issue in the `Blocked by` line) or an assignee; first in map order wins.
- **Claim**: `gh api repos/{owner}/{repo}/issues/<n>/assignees -f 'assignees[]=santiagobartolini'`, the session's first write. The REST API has no `@me`, and the current-user endpoint isn't reachable from cloud sessions.
- **Resolve**: comment with the answer, close the issue (both as above), then append a context pointer (gist + link) to the map's Decisions-so-far: read the map's current body and send it back with the pointer appended, using `gh api -X PATCH repos/{owner}/{repo}/issues/<map> -F body=@-`.

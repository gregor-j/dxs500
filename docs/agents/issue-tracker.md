# Issue tracker: GitHub

Issues, specs, and tickets live in this repository's GitHub Issues. Use the `gh` CLI. Infer the repository from the Git remote.

## Operations

- Publish a spec or ticket: `gh issue create --title "..." --body-file -` with a heredoc for the body.
- Read a ticket: `gh issue view <number> --json number,title,body,labels,comments`.
- List tickets: `gh issue list --state open --json number,title,body,labels`; use label filters when needed.
- Comment: `gh issue comment <number> --body "..."`.
- Change labels: `gh issue edit <number> --add-label "..." --remove-label "..."`.
- Close: `gh issue close <number> --comment "..."`.

Write issue titles, descriptions, and comments in English.

## Pull requests as a triage surface

**PRs as a request surface: no.**

Pull requests deliver changes; they are not part of the issue triage queue. Follow `CONTRIBUTING.md` for the pull request process.

## Parent tickets and dependencies

Use GitHub sub-issues to link child tickets to a parent. Use native issue dependencies for blocking edges. Discover supported CLI commands with `gh issue --help`; use `gh api` when needed.

- Link a child: `gh api --method POST repos/{owner}/{repo}/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`.
- Add a blocker: `gh api --method POST repos/{owner}/{repo}/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`.
- Obtain a database ID: `gh api repos/{owner}/{repo}/issues/<number> --jq .id`. This is not the issue number or node ID.

If these features are unavailable, record `Part of #<parent>` and `Blocked by: #<number>` in child bodies, and list children in the parent body. Report the limitation explicitly.

## Wayfinding

Use a `wayfinder:map` issue for the map and linked child issues for decision tickets. Create wayfinder labels only when needed: `wayfinder:map` and `wayfinder:<type>` (`research`, `prototype`, `grilling`, or `task`).

An eligible frontier ticket is open, unassigned, and has no open blockers. Select the first eligible child in map order. Claim it with `gh issue edit <number> --add-assignee @me`. To resolve it, comment with the answer, close it, and add a summary and link to the map's Decisions-so-far section.

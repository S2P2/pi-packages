# Issue tracker: GitHub

Issues and specs live in GitHub Issues for `S2P2/pi-packages`.
Use the `gh` CLI from this checkout; it infers the repository from the remote.

## Operations

- Create: `gh issue create --title "..." --body-file <file>`
- Read ticket: `gh issue view <number> --comments`
- List: `gh issue list --state open --json number,title,body,labels,comments`
- Comment: `gh issue comment <number> --body-file <file>`
- Label: `gh issue edit <number> --add-label "..."` or `--remove-label "..."`
- Close: `gh issue close <number> --comment "..."`

When a skill says "publish to the issue tracker", create a GitHub issue.
When it says "fetch the relevant ticket", read the issue and its comments.

## Pull requests as a triage surface

**PRs as a request surface: no.**

## Wayfinding operations

The canonical map is an issue labelled `wayfinder:map`.
Its tickets are native GitHub sub-issues labelled `wayfinder:research`,
`wayfinder:prototype`, `wayfinder:grilling`, or `wayfinder:task`.
Refer to issues by linked title in human-facing text.

Create all issues before wiring relationships. GitHub relationship endpoints
use the issue's numeric database `id`, not its issue number or GraphQL id:

```bash
gh api repos/S2P2/pi-packages/issues/<child-number> --jq .id
gh api --method POST repos/S2P2/pi-packages/issues/<map-number>/sub_issues \
  -F sub_issue_id=<child-database-id>
gh api --method POST repos/S2P2/pi-packages/issues/<blocked-number>/dependencies/blocked_by \
  -F issue_id=<blocker-database-id>
```

Claim a ticket before work with `gh issue edit <number> --add-assignee @me`.
An open, assigned issue is claimed. For the frontier, list native children,
keep open/unassigned issues, then exclude any with an open blocker:

```bash
gh api --paginate repos/S2P2/pi-packages/issues/<map-number>/sub_issues \
  --jq '.[] | select(.state == "open" and (.assignees | length) == 0) | {number,title,html_url}'
gh api repos/S2P2/pi-packages/issues/<candidate-number>/dependencies/blocked_by \
  --jq '[.[] | select(.state != "closed")] | length'
```

Take the first eligible child in the map's native sub-issue order.
Resolve by posting an answer comment with asset pointers, closing the ticket,
and adding one linked-title gist to the map's Decisions-so-far. The detailed
answer lives only on the ticket; the map does not list open tickets.
Concurrent resolvers must serialize map-body updates and fetch the latest
body immediately before editing so another resolver's pointers survive.

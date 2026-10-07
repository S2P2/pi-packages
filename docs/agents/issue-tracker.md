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

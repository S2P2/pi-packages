# Changesets

Once packages exist, run `npm run changeset` for releasable changes. Select the affected packages, version bump, and write a user-facing summary. Commit the generated Markdown file alongside the change.

Tooling-only changes at the private workspace root do not need a changeset.

- `npm run changeset:status`: inspect pending releases.
- `npm run version:packages`: consume changesets, update package versions and changelogs, and refresh the lockfile.
- `npm run release`: run checks/tests, then publish changed packages and create release tags. Requires npm credentials.

Packages version independently; private packages are neither versioned nor tagged. The base branch is `main`. Publishing defaults to restricted access; choose public access explicitly before publishing public packages.

See the [Changesets documentation](https://changesets.dev/).

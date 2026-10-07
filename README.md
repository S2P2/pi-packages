# pi-packages

Personal [Pi](https://github.com/earendil-works/pi) packages: extensions, skills, prompts, themes, and shared libraries.

## Ongoing design

The [Pi subagent specification wayfinding map](https://github.com/S2P2/pi-packages/issues/1) tracks the current planning effort. Continue from the map's native child issues and blocking relationships; GitHub issue resolutions are the canonical decisions, not a local draft spec.

The [design baseline](https://github.com/S2P2/pi-packages/issues/2) captures agreements from the initial discussion. Supporting notes live in `docs/research/`; background research findings are linked from their tickets on `research/pi-subagent-*` branches. No subagent package implementation has started.

## Setup

Requires Node.js 22.19+ and npm 11.

```sh
npm install
npm run check
npm test
```

The root is a private npm workspace. `npm install` enables Husky's pre-commit hook, which runs formatting/lint checks and TypeScript checks without modifying or restaging files. Run `npm run check:fix` to apply safe formatting/lint fixes, then review and stage them yourself.

## Layout

Follows Pi's npm monorepo organization, without its core-runtime machinery:

```text
packages/
  <package>/
    package.json
    src/
    test/
    README.md
    CHANGELOG.md
    tsconfig.json
    docs/             # optional
    skills/           # optional
    prompts/          # optional
    themes/           # optional
    dist/             # generated, if compiled
```

No package is created yet. Shared tooling lives at the root; shared runtime code can become a separate workspace package.

## Development

- `npm run check`: Biome formatting/lint checks and TypeScript checks.
- `npm run check:fix`: apply safe Biome fixes.
- `npm run typecheck`: type-check root tooling and package source/tests.
- `npm test`: run package `test/**/*.test.ts` files with Vitest.
- `npm run test:watch`: watch tests.

Tests allow an empty suite while this is only a scaffold. Remove `--passWithNoTests` once the first package adds tests. Root Vitest configuration uses a shared Node environment; introduce package-specific Vitest projects if packages need different settings.

Each package's TypeScript config should extend `../../tsconfig.base.json`. Compiled packages should build only `src/` into `dist/`; package-local build/prepack scripts must generate any artifacts required for publication. There is deliberately no build setup until a package's source-vs-compiled distribution is chosen.

Loadable Pi packages must declare a `pi` resource manifest pointing at their actual entrypoints. Direct TypeScript loading needs no build; compiled packages must point at generated JavaScript and include every runtime helper/resource in their npm `files` allowlist.

Host-provided Pi imports belong in `peerDependencies` with `"*"`, not runtime dependencies. Ordinary third-party runtime libraries belong in `dependencies`. Package names/scopes, licensing, and distribution formats remain to be chosen.

Install individual package directories with `pi install ./packages/<package>`. This private workspace root has no aggregate `pi` manifest and is not itself a loadable Pi bundle.

## Releases

[Changesets](.changeset/README.md) manages independent package versions and package-local changelogs. Make the repository's initial commit on `main` before running `changeset:status`; Changesets needs an existing base-branch commit to compare against.

```sh
npm run changeset          # record a releasable package change
npm run changeset:status   # review pending releases
npm run version:packages   # update versions, changelogs, and lockfile
# Review and commit the version changes before publishing.
npm run release           # checks/tests, then npm publication and Git tags
# Push the release commit and tags separately.
```

Publishing requires npm authentication and an explicit access decision. Changesets currently uses its safe `restricted` default; switch to `public` globally or through a package's `publishConfig.access` when appropriate. Set `private: true` on packages that should never publish.

No automatic release workflow, package scope, or license has been selected.

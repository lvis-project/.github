# lvis-project — Governance

This file records the org-level rules that are **not** visible in any single
repository's code. Anything a command can answer is left to the command; a table
copied from live state is a second source of truth that starts rotting the day
it is written, and most of what used to be in this file had.

## Tag policy — enforced, not conventional

The org is on the GitHub **Team** plan and rulesets are active. Tag immutability
is enforced by GitHub, not by convention:

| Ruleset target | Rule |
|---|---|
| `refs/tags/v*` | `update` blocked — a tag cannot be re-pointed |
| `refs/tags/v*` | `deletion` blocked — a tag cannot be removed |

Active in `.github`, `lvis-app`, `lvis-plugin-sdk`, and every `lvis-plugin-*`
release repo. Verify with:

```bash
gh api repos/lvis-project/<repo>/rulesets --jq '.[] | "\(.name) \(.target) \(.enforcement)"'
```

Two consequences worth stating plainly:

1. **There are no moving major tags.** `update` is blocked, so `v1` cannot be
   re-pointed at a newer commit. Historical `v1` / `v1.0.0` tags still exist in
   `.github` and in the SDK; they are frozen where they are. To ship a change,
   cut a new version — never try to move a tag.
2. **A release tag is the release.** The SDK and plugin publish pipelines take
   the tag as the single source of truth and re-derive everything from it
   (annotated, peeled commit reachable from `origin/main`, version matching the
   manifest). An immutable tag is what makes that derivation trustworthy.

## Branch protection

`.github` main requires a pull request and a passing `lint` check, blocks force
pushes and deletions, and applies to admins. `lvis-marketplace` and
`lvis-plugin-git` carry branch rulesets of their own. The rest of the org relies
on repository branch protection; check a specific repo with:

```bash
gh api repos/lvis-project/<repo>/branches/main/protection --jq '{checks: .required_status_checks.contexts, force: .allow_force_pushes.enabled}'
```

## How plugin repos reference the shared workflows

Callers name the reusable workflows by branch:

```yaml
uses: lvis-project/.github/.github/workflows/plugin-ci.yml@main
uses: lvis-project/.github/.github/workflows/validate-plugin-manifest.yml@main
uses: lvis-project/.github/.github/workflows/marketplace-publish.yml@main
```

**`@main`, deliberately, and not a commit SHA.** The publisher runner group is
workflow-restricted, and GitHub matches that allow-list against the exact ref a
caller writes. A SHA in the caller therefore stores the same decision twice —
once in a reviewed workflow file, once in an org setting no pull request sees —
and the two drifting is not a loud failure. The job is simply never scheduled:
it queues until the 24-hour limit expires, with the runner reported online and
idle and no red check anywhere. In 2026-08 that cost every plugin several days
of silently broken publishing, twice over: once when a history rewrite in this
repo orphaned the pinned commits, and again when repointing the pins moved the
callers out of the allow-list.

What actually guards the credential is unchanged by using a branch here:

- `main` in this repo is protected, so a revision arrives only through a
  reviewed pull request that passes CI.
- Each caller's `publish.yml` is `repository_dispatch`-only, so GitHub always
  loads it from that repo's protected default branch — a tag can never supply
  the workflow that receives the key.
- The publisher runner group admits only `marketplace-publish.yml` onto the
  publisher host, which is the only tier that receives the Marketplace API key.
- The reusable workflow re-derives and verifies the tag before the key is in
  scope.

Consequently the runner-group allow-lists name branches, not commits:
`lvis-oracle-publisher` allows
`.../marketplace-publish.yml@refs/heads/main`, and `lvis-oracle-deploy` has
always named its entries the same way. **If you ever pin a caller to a SHA
again, the publisher allow-list must be edited in the same change or publishing
stops without an error.** That is the trap this policy exists to remove.

Every self-hosted job in the shared workflows carries a `timeout-minutes`, so an
unschedulable job fails visibly instead of waiting out the queue limit.

## How plugin repos consume the SDK

`@lvis/plugin-sdk` is a `devDependency` resolved from a Git ref:

```json
"@lvis/plugin-sdk": "github:lvis-project/lvis-plugin-sdk#vMAJOR.MINOR.PATCH"
```

There are **no submodules** anywhere in the org; earlier revisions of this file
described a `plugin-sdk/` submodule layout that no longer exists. Pin to a
release tag rather than a bare commit — a tag survives a history rewrite in the
SDK, and a bare commit does not, which is the same failure that took out CI here.

Pins are deliberately per-repo: a plugin adopts a new SDK major when its author
is ready, and the version floor a plugin needs from the host lives in
`requires.minAppVersion` in its `plugin.json`. To see where every repo currently
sits:

```bash
gh api repos/lvis-project/<repo>/contents/package.json --jq '.content' \
  | base64 -d | grep '@lvis/plugin-sdk'
```

## Repo inventory

Enumerate it; do not read it from here:

```bash
gh repo list lvis-project --limit 100
```

The shape is stable even when the list is not:

- `lvis-app` — the Electron host. Owns the plugin manifest schema and the public
  plugin contract; every other contract in the org is generated from it.
- `lvis-plugin-sdk` — type-only SDK, mirrored from the host by sync scripts and
  gated by drift checks. Never hand-edited.
- `lvis-marketplace` — validates and signs uploaded artifacts, serves the catalog.
- `lvis-plugin-*` — plugins. Each carries a thin CI/publish caller pointing at
  the shared workflows above.
- `.github` — this repo: shared reusable workflows, the org workflow template,
  and this file.

## CI architecture

- **Shared reusable workflows** live in `.github/workflows/` here:
  `plugin-ci.yml` (typecheck, build, test), `validate-plugin-manifest.yml`
  (manifest schema plus the version-bump guard that keeps the published build
  from going stale), and `marketplace-publish.yml` (candidate build on a
  GitHub-hosted runner, upload on the isolated publisher tier).
- **Consumers** are `lvis-plugin-*` repos, via thin callers in their own
  `.github/workflows/`.
- **Scaffold** for a new plugin: `lvis-project/lvis-plugin-template`, and the org
  workflow template in `.github/workflow-templates/`.
- **Runner groups**: `lvis-oracle-ci` for shared CI, `lvis-oracle-deploy` for
  marketplace deploys, `lvis-oracle-publisher` for the publish step alone. The
  publisher tier is deliberately not part of the other two.

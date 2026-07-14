# `publish.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The logic is identical; npm scope defaults differ by org (`@frmscoe` vs `@tazama-lf`).
>
> This is a **central reusable workflow**. It is **not** copied into consumer repos by `sync-workflows.yml`. Package repos receive a thin caller stub via the release bootstrap process instead. The **push to main** trigger lives on the caller stub, not on this central file.

## Purpose

Publishes an npm package to GitHub Packages. Dist-tag is derived from `package.json` version:

- Prerelease (e.g. `1.2.3-rc.1`) → `rc`
- Clean semver (e.g. `1.2.3`) → `latest`

Companion gate: [`version-check.yml`](version-check.md) blocks merging prerelease versions to `main`, so stable publishes stay on clean semver.

---

## Trigger

### Central reusable workflow

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Invoked by a caller stub |
| `workflow_dispatch` | Manual (must run from publish/target branch) |

### Typical caller stub (in each package repo)

| Event | Conditions |
|-------|-----------|
| `push` | target branch (`main`); paths: `package.json`, `package-lock.json` |
| `workflow_dispatch` | manual from target branch |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Node version | `22` (overridable) |
| Registry | `https://npm.pkg.github.com/` |
| Permissions | `packages: write`, `contents: read`, `pull-requests: read` |

---

## Inputs

| Input | Default | Purpose |
|-------|---------|---------|
| `node_version` | `22` | Node.js version |
| `package_scope` | `@frmscoe` | Primary npm scope for setup-node |
| `run_build` | `true` | Run build before publish |
| `build_command` | `npm run build` | Build command when `run_build` is true |
| `skip_if_already_published` | `true` | Skip publish if `name@version` already exists |
| `publish_branch` | `main` | Branch the workflow must run from |

### Useful repo / org variables

`NODE_VERSION`, `PACKAGE_SCOPE`, `RUN_BUILD`, `BUILD_COMMAND`, `SKIP_IF_ALREADY_PUBLISHED`, `TARGET_BRANCH`

Auth is configured for both `@frmscoe` and `@tazama-lf` registries so internal cross-org deps resolve.

---

## Jobs

### `build-and-publish`

1. Validate the workflow is running from the publish/target branch
2. Checkout with `GH_TOKEN_LIB`
3. Setup Node.js + npm auth
4. Resolve package name, version, and dist-tag from `package.json`
5. Optionally skip if already published
6. `npm ci` (or `npm install` without lockfile)
7. Optional build
8. `npm publish --tag {rc|latest}`
9. Resolve source PR (best effort) and Slack notify (skipped when already published)

---

## Required Secrets

| Secret | Scope | Required | Purpose |
|--------|-------|----------|---------|
| `GH_TOKEN_LIB` | org | required | npm auth and checkout |
| `SLACK_WEBHOOK_URL` | org | optional | Slack notification |

---

## Caller stub (installed by bootstrap, not by sync)

```yaml
# .github/workflows/publish.yml (in package repos - managed centrally)
on:
  push:
    branches: [main]   # TARGET_BRANCH
    paths:
      - 'package.json'
      - 'package-lock.json'
  workflow_dispatch:

jobs:
  publish:
    uses: frmscoe/workflows/.github/workflows/publish.yml@v1
    with:
      node_version: ${{ vars.NODE_VERSION || '22' }}
      package_scope: ${{ vars.PACKAGE_SCOPE || '@frmscoe' }}
      run_build: ${{ vars.RUN_BUILD != 'false' }}
      build_command: ${{ vars.BUILD_COMMAND || 'npm run build' }}
      skip_if_already_published: ${{ vars.SKIP_IF_ALREADY_PUBLISHED != 'false' }}
      publish_branch: ${{ vars.TARGET_BRANCH || 'main' }}
    secrets: inherit
```

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | **Not synced** — excluded from the sync bundle |
| Package / library repos | Caller stub installed via bootstrap when `install_publish=true` |

Rule Docker builds use [`package-rule.yml`](package-rule.md) / [`package-rule-rc.yml`](package-rule-rc.md), not this publish workflow.

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e…` | v6.0.2 |
| `actions/setup-node` | `53b83947…` | v6.3.0 |

---

## frmscoe vs tazama-lf differences

| Property | `frmscoe/workflows` | `tazama-lf/workflows` |
|----------|--------------------|-----------------------|
| Default `package_scope` | `@frmscoe` | `@tazama-lf` (when overridden in caller) |
| Published to | org GitHub Packages | org GitHub Packages |

Logic is shared; callers set `PACKAGE_SCOPE` per org.

---

## Known Limitations / Notes

- Must run from the target branch after the release PR merges
- Does not create GitHub git tags — that is [`release.yml`](release.md)
- `dependabot[bot]` actors are excluded
- Set `RUN_BUILD=false` for packages that publish without a build step

---

## Repository Overrides

| Variable | When to override |
|----------|------------------|
| `RUN_BUILD=false` | Packages with no build script |
| `PACKAGE_VERSION_POLICY=independent` | Independent libs (affects train/release pairing, not publish itself) |

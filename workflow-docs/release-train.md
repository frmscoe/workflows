# `release-train.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The logic is identical.
>
> This is a **central reusable workflow**. It is **not** copied into consumer repos by `sync-workflows.yml`. Package repos receive a thin caller stub via the release bootstrap process instead.

## Purpose

Prepares a **platform release PR** for repositories that have `package.json`:

1. Validates / resolves the platform release version (e.g. `4.0.0`)
2. Applies version policy (`aligned` or `independent`)
3. Resolves internal `@frmscoe` / `@tazama-lf` **rc** dependencies to stable versions
4. Updates `package.json` (and regenerates `package-lock.json` when present)
5. Force-pushes branch `release/v{platform}` and opens a PR to the target branch (usually `main`)

For repos **without** `package.json`, use [`dev-to-main-pr.yml`](dev-to-main-pr.md) instead.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Invoked by a caller stub in the target repo |
| `workflow_dispatch` | Manual run from the **source branch** (usually `dev`) |

Must run from the source branch. Running from `main` fails the branch guard.

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Node version | `22` (overridable) |
| Permissions | `contents: write`, `pull-requests: write` |

---

## Inputs

| Input | Default | Purpose |
|-------|---------|---------|
| `version` | `""` | Platform release version. Blank → `RELEASE_VERSION` var, then stable `package.json` version |
| `version_policy` | `aligned` | `aligned` sets `package.json` to the platform version; `independent` keeps the repo’s own stable semver |
| `source_branch` | `dev` | Branch the workflow must run from |
| `target_branch` | `main` | PR base branch |
| `package_scope` | `@frmscoe` | npm scope for setup-node |
| `node_version` | `22` | Node.js version |
| `internal_scopes_regex` | `^@(tazama-lf\|frmscoe)/` | Which dependency scopes get rc → stable rewriting |
| `version_mismatch_behavior` | `fail` | `fail` or `warn` when aligned policy sees a mismatch |
| `reviewer` | `""` | Optional PR reviewer |

### Useful repo / org variables

`RELEASE_VERSION`, `PACKAGE_VERSION_POLICY`, `SOURCE_BRANCH`, `TARGET_BRANCH`, `PACKAGE_SCOPE`, `NODE_VERSION`, `INTERNAL_SCOPES_REGEX`, `VERSION_MISMATCH_BEHAVIOR`, `RELEASE_REVIEWER`

---

## Jobs

### `prepare-release`

1. Validate the workflow is running from the source branch
2. Checkout source branch with `GH_TOKEN_LIB`
3. Set up Node.js + GitHub Packages auth for `@frmscoe` and `@tazama-lf`
4. Resolve platform version and package version to set
5. Rewrite internal rc dependencies:
   - Exact pinned rc (`1.2.3-rc.1`) → `npm view … dist-tags.latest` stable
   - Range with rc (`^1.2.3-rc.1`) → strip prerelease (`^1.2.3`), still requires stable latest published
6. Set `package.json` version
7. Regenerate lockfile with `npm install --package-lock-only` when `package-lock.json` exists
8. Commit and force-push `release/v{platform}`
9. Open PR to target branch titled `release: v{platform}` (idempotent if PR already exists)

---

## Required Secrets

| Secret | Scope | Required | Purpose |
|--------|-------|----------|---------|
| `GH_TOKEN_LIB` | org | required | Checkout, npm auth, push branch, create PR |
| `GH_USERNAME` | org | optional | Fallback reviewer when `RELEASE_REVIEWER` is not set |

---

## Caller stub (installed by bootstrap, not by sync)

```yaml
# .github/workflows/release-train.yml (in package repos - managed centrally)
on:
  workflow_dispatch:
    inputs:
      version:
        description: "Platform release version"
        required: false
        default: ""

jobs:
  release-train:
    uses: frmscoe/workflows/.github/workflows/release-train.yml@v1
    with:
      version: ${{ inputs.version || vars.RELEASE_VERSION || '' }}
      version_policy: ${{ vars.PACKAGE_VERSION_POLICY || 'aligned' }}
      source_branch: ${{ vars.SOURCE_BRANCH || 'dev' }}
      target_branch: ${{ vars.TARGET_BRANCH || 'main' }}
      package_scope: ${{ vars.PACKAGE_SCOPE || '@frmscoe' }}
      node_version: ${{ vars.NODE_VERSION || '22' }}
      internal_scopes_regex: ${{ vars.INTERNAL_SCOPES_REGEX || '^@(tazama-lf|frmscoe)/' }}
      version_mismatch_behavior: ${{ vars.VERSION_MISMATCH_BEHAVIOR || 'fail' }}
      reviewer: ${{ vars.RELEASE_REVIEWER || '' }}
    secrets: inherit
```

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | **Not synced** — excluded from the sync bundle |
| Package / rule-package repos | Caller stub installed via bootstrap when `install_release_train=true` |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e…` | v6.0.2 |
| `actions/setup-node` | `53b83947…` | v6.3.0 |

---

## Known Limitations / Notes

- Requires `package.json`. Non-code repos must use [`dev-to-main-pr.yml`](dev-to-main-pr.md)
- Upstream internal packages must already be published at a **stable** latest before rc dependency rewrite succeeds
- Force-pushes the release branch — expected for re-runs of the same platform version
- After merge: [`publish.yml`](publish.md) publishes npm; [`release.yml`](release.md) creates the GitHub tag
- `dependabot[bot]` actors are excluded

---

## Repository Overrides

| Policy | Typical use |
|--------|-------------|
| `aligned` | Services, rules, apps that track the platform version |
| `independent` | Shared libraries that keep their own semver while still naming the PR `release/v{platform}` |

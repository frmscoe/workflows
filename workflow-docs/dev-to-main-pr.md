# `dev-to-main-pr.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The logic is identical in both orgs.
>
> This file is a **central reusable workflow**. It is **not** copied into consumer repos by `sync-workflows.yml`. Non-code repos receive a thin caller stub via the release bootstrap process instead.

## Purpose

Opens a pull request from the source branch (typically `dev`) to the target branch (typically `main`) **without modifying `package.json` or lockfiles**.

Use this for:

- Documentation and config repos with no npm package
- Repositories that take part in a platform release wave but do not need dependency bumps or version alignment
- Any repo where `release-train.yml` would fail because `package.json` is absent

For package/library repos, use [`release-train.yml`](release-train.md) instead — that workflow bumps versions, resolves internal rc dependencies to stable, and opens a `release/vX.Y.Z` branch PR.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Invoked by a caller stub in the target repo |
| `workflow_dispatch` | Manual run (from the central workflows repo, or via a caller) |

The caller stub in each target repo is normally triggered with `workflow_dispatch` from the **source branch**.

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Typical duration | ~30 seconds |
| Concurrency | none |
| Permissions | `contents: write`, `pull-requests: write` |

---

## Inputs

| Input | Default | Purpose |
|-------|---------|---------|
| `source_branch` | `dev` | Head branch for the PR |
| `target_branch` | `main` | Base branch for the PR |
| `release_version` | `""` | Optional version for the PR title (`release: v4.0.0`) |
| `reviewer` | `""` | Optional GitHub username to request as reviewer |

Inputs may also be supplied via repo/org Actions variables:

| Variable | Purpose |
|----------|---------|
| `SOURCE_BRANCH` | Overrides default source branch |
| `TARGET_BRANCH` | Overrides default target branch |
| `RELEASE_VERSION` / `DEFAULT_RELEASE_VERSION` | Version used in the PR title when no input is passed |
| `RELEASE_REVIEWER` | Preferred reviewer for the PR |

---

## Jobs

### `create-pr`

**Steps:**

1. Resolve source/target branches and optional release version from inputs or repo/org variables
2. Build PR title:
   - With version: `release: v4.0.0`
   - Without version: `release: merge dev → main`
3. Check for an existing open PR with the same head/base — if one exists, exit successfully without creating a duplicate
4. Optionally assign a reviewer (`RELEASE_REVIEWER` variable, or `GH_USERNAME` secret as fallback)
5. Create the PR with a standard release-preparation body

---

## Required Secrets

| Secret | Scope | Required | Purpose |
|--------|-------|----------|---------|
| `GH_TOKEN_LIB` | org | recommended | PR creation with elevated permissions |
| `GH_USERNAME` | org | optional | Fallback reviewer when `RELEASE_REVIEWER` is not set |

Falls back to `github.token` when `GH_TOKEN_LIB` is not provided. Org-level `GH_TOKEN_LIB` is required for the release orchestration scripts that trigger this workflow across many repos.

---

## Caller stub (installed by bootstrap, not by sync)

```yaml
# .github/workflows/dev-to-main-pr.yml (in non-code repos - managed centrally)
on:
  workflow_dispatch:
    inputs:
      release_version:
        description: "Optional release version"
        required: false
        default: ""

jobs:
  dev-to-main-pr:
    uses: frmscoe/workflows/.github/workflows/dev-to-main-pr.yml@v1
    with:
      source_branch: ${{ vars.SOURCE_BRANCH || 'dev' }}
      target_branch: ${{ vars.TARGET_BRANCH || 'main' }}
      release_version: ${{ inputs.release_version || vars.RELEASE_VERSION || vars.DEFAULT_RELEASE_VERSION || '' }}
      reviewer: ${{ vars.RELEASE_REVIEWER || '' }}
    secrets: inherit
```

Replace `frmscoe` / `@v1` with the appropriate org and workflows ref for `tazama-lf` repos when applicable.

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | **Not synced** — excluded from the sync bundle |
| Non-code / docs / config repos | Caller stub installed via bootstrap when `install_dev_pr=true` in the release manifest |

Rule repos use [`release-train.yml`](release-train.md) (via caller) when they participate in a package release; they do not receive this workflow from sync.

---

## Dependencies (pinned actions)

None — this workflow uses the runner `gh` CLI and does not invoke third-party Actions.

---

## frmscoe vs tazama-lf differences

| Property | `frmscoe/workflows` | `tazama-lf/workflows` |
|----------|--------------------|-----------------------|
| Workflow logic | identical | identical |
| Caller stub org ref | `frmscoe/workflows@v1` | `tazama-lf/workflows@v1` |

---

## Known Limitations / Notes

- Does not create a release branch — the PR is directly from source → target
- Does not bump versions or regenerate lockfiles
- Idempotent: re-running when a PR already exists is safe (no duplicate PRs)
- `dependabot[bot]` actors are excluded
- `release-train.yml` fails when `package.json` is missing; use this workflow for non-code repos instead

---

## Repository Overrides

| Repository type | Workflow used |
|-----------------|---------------|
| npm package / rule-package | `release-train.yml` |
| non-code (docs, config) | `dev-to-main-pr.yml` |

Selection is configured per repo in the release manifest (`install_dev_pr` / `install_release_train` columns), not by sync.

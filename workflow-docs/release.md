# `release.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The logic is identical.
>
> This is a **central reusable workflow**. It is **not** copied into consumer repos by `sync-workflows.yml`. Target repos receive a thin caller stub via the release bootstrap process instead.
>
> **Note:** This replaces the older milestone-driven auto-bump release workflow. It creates a platform GitHub tag/release (`vX.Y.Z`); it does not publish npm packages (see [`publish.yml`](publish.md)) and does not bump `package.json`.

## Purpose

Creates a **platform GitHub release and tag** after a release PR has been merged to the target branch (usually `main`).

- Tag version comes from the platform release version (`RELEASE_VERSION` / `tag_version`), not from conventional-commit auto-bumping
- npm `package.json` version is recorded in the release notes when present
- Repos without `package.json` fall back to `DEFAULT_RELEASE_VERSION` / `default_version`

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Invoked by a caller stub in the target repo |
| `workflow_dispatch` | Manual run from the **target branch** |
| `repository_dispatch` | types: `[release]` — kept for backward compatibility only |

Normal releases should use the caller stub (`workflow_dispatch` / orchestration scripts), not `repository_dispatch`.

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Typical duration | ~1–2 min |
| Permissions | `contents: write`, `actions: read` |

---

## Inputs

| Input | Default | Purpose |
|-------|---------|---------|
| `tag_version` | `""` | Platform GitHub release version, e.g. `4.0.0` |
| `default_version` | `4.0.0` | Fallback when there is no `package.json` and no tag input |
| `expected_version` | `""` | Deprecated alias for `tag_version` |
| `version_policy` | `aligned` | `aligned` requires `package.json` stable version to match the platform tag; `independent` allows a different npm version |
| `version_mismatch_behavior` | `fail` | `fail` or `warn` when aligned policy mismatches |
| `release_branch` | `main` | Branch the release must run from |
| `milestone_number` | `""` | Optional milestone to include in release notes |

### Useful repo / org variables

`RELEASE_VERSION`, `DEFAULT_RELEASE_VERSION`, `PACKAGE_VERSION_POLICY`, `VERSION_MISMATCH_BEHAVIOR`, `TARGET_BRANCH`, `RELEASE_MILESTONE_NUMBER`

---

## Jobs

### `release`

1. Validate the workflow is running from the release/target branch (skipped for `repository_dispatch`)
2. Checkout the release branch with full history
3. Resolve platform tag version (`vX.Y.Z`) from inputs → variables → `package.json` → default
4. Under `aligned` policy, fail/warn if `package.json` stable version does not match the platform tag
5. Refuse to recreate an existing git tag or GitHub release
6. Optionally fetch milestone details
7. Generate a conventional-commit changelog since the previous tag
8. Upload changelog artifact
9. `gh release create` with the generated notes
10. Resolve source PR (best effort) and send Slack notification when configured

---

## Required Secrets

| Secret | Scope | Required | Purpose |
|--------|-------|----------|---------|
| `SLACK_WEBHOOK_URL` | org | optional | Slack notification after release create |

Uses the auto-provided `GITHUB_TOKEN` for release creation.

---

## Caller stub (installed by bootstrap, not by sync)

```yaml
# .github/workflows/release.yml (in target repos - managed centrally)
on:
  workflow_dispatch:
    inputs:
      tag_version:
        description: "Platform release version, e.g. 4.0.0"
        required: false
        default: ""

jobs:
  release:
    uses: frmscoe/workflows/.github/workflows/release.yml@v1
    with:
      tag_version: ${{ inputs.tag_version || vars.RELEASE_VERSION || '' }}
      default_version: ${{ vars.DEFAULT_RELEASE_VERSION || '4.0.0' }}
      version_policy: ${{ vars.PACKAGE_VERSION_POLICY || 'aligned' }}
      version_mismatch_behavior: ${{ vars.VERSION_MISMATCH_BEHAVIOR || 'fail' }}
      release_branch: ${{ vars.TARGET_BRANCH || 'main' }}
      milestone_number: ${{ vars.RELEASE_MILESTONE_NUMBER || '' }}
    secrets: inherit
```

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | **Not synced** — excluded from the sync bundle |
| Release-wave repos | Caller stub installed via bootstrap when `install_release=true` |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e…` | v6.0.2 |
| `actions/upload-artifact` | `bbbca2dd…` | — |

---

## Known Limitations / Notes

- Does **not** update `CHANGELOG.md` or a `VERSION` file in the repo — notes live on the GitHub release
- Does **not** publish npm or Docker images
- Fail-closed if the tag or release already exists (safe for retries after partial success)
- `dependabot[bot]` actors are excluded
- Prefer caller + orchestration scripts over `repository_dispatch`

---

## Repository Overrides

| Repository type | Notes |
|-----------------|-------|
| Package / rule-package | Tag usually matches platform version; npm version recorded in notes |
| Non-code | Uses `default_version` / `DEFAULT_RELEASE_VERSION` for the platform tag |

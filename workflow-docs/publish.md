# `publish.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The logic is identical but the npm scope differs - this variant publishes under `@frmscoe`, not `@tazama-lf`.
>
> For the `tazama-lf` variant, see [`workflow-docs/publish.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/publish.md).

## Purpose

Publishes an npm package to GitHub Packages under the `@frmscoe` scope. Determines the dist-tag from the version in `package.json`:

- Prerelease version (e.g. `1.2.3-rc.1`) → `rc` dist-tag (triggered manually from `dev`)
- Clean semver (e.g. `1.2.3`) → `latest` dist-tag (fires automatically on `push: main`)

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[main]` |
| `workflow_dispatch` | manual |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Node version | `20` |
| npm scope | `@frmscoe` |
| Registry | `https://npm.pkg.github.com/` |
| Permissions | `packages: write`, `contents: read` |

---

## Jobs

### `build-and-publish`

1. `actions/checkout@v4` - with `GH_TOKEN_LIB` token
2. `actions/setup-node@v4` - Node 20, registry `https://npm.pkg.github.com/`, scope `@frmscoe`
3. Set up npm authentication (writes `_authToken` to `.npmrc`)
4. `npm ci`
5. `npm run build`
6. Publish - reads version from `package.json`; publishes under `rc` if prerelease, otherwise `latest`
7. Notify Slack

---

## Required Secrets

| Secret | Scope | Purpose |
|--------|-------|---------|
| `GH_TOKEN_LIB` | org | npm authentication and checkout |
| `SLACK_WEBHOOK_URL` | org | Slack notification |

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | Receives this file |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | tag ref `v4` | - |
| `actions/setup-node` | tag ref `v4` | - |

---

## frmscoe vs tazama-lf differences

| Property | `frmscoe/workflows` | `tazama-lf/workflows` |
|----------|--------------------|-----------------------|
| npm scope | `@frmscoe` | `@tazama-lf` |
| Published to | `frmscoe` GitHub Packages | `tazama-lf` GitHub Packages |

---

## Known Limitations / Notes

- `version-check.yml` blocks merging a prerelease version to `main`, ensuring only clean semver versions are published as `latest`.
- `dependabot[bot]` actors are excluded.

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | _(all synced repos use this version)_ |

# `version-check.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. Behaviour is identical.
>
> For rule repos, this is a PR gate that pairs with stable [`package-rule.yml`](package-rule.md). For npm package repos it pairs with [`publish.yml`](publish.md).

## Purpose

Blocks merging a PR to `main` when `package.json` still contains a **prerelease** version suffix (e.g. `1.2.3-rc.1`).

Only clean semver (`X.Y.Z`) should reach `main`, so that:

- npm `latest` publishes stay stable
- Docker `:latest` rule images stay stable

Developers (or [`release-train.yml`](release-train.md)) must strip the prerelease suffix before the PR to `main` can pass.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `pull_request` | branches: `[main]` |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Permissions | `contents: read` |

---

## Jobs

### `check-version`

1. Checkout the PR head
2. Fail if `package.json` is missing
3. Fail if `.version` is missing
4. Fail if version contains `-` (any prerelease / build suffix pattern used here)
5. Otherwise pass

---

## Required Secrets

None (uses default `GITHUB_TOKEN`).

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | **Synced** — included in the sync bundle so rule repos get the PR gate |

Library / platform package repos that are not in the rule sync list should keep this file via their own bootstrap / repo setup.

Enable as a **required status check** on `main` for maximum effect.

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e…` | v6.0.2 |

---

## Known Limitations / Notes

- Does not publish, tag, or edit `package.json` — gate only
- Does not distinguish prerelease kinds (`rc`, `beta`, etc.); any `-` in the version fails
- Companion failure mode in [`package-rule.yml`](package-rule.md): stable Docker build also refuses prerelease versions if a gate was bypassed

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | Same check in all synced rule repos |

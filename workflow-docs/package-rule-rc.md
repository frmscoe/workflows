# `package-rule-rc.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The reusable workflow logic is identical to the stable variant, but triggers on `push: dev` to produce an `:rc` Docker tag. Caller stubs reference `frmscoe/workflows` and pass `rule_org: "frmscoe"`.
>
> For the full workflow logic documentation and the `tazama-lf` variant, see [`workflow-docs/package-rule-rc.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/package-rule-rc.md).

## Purpose

Reusable workflow that builds and pushes an `:rc` Docker image for a `frmscoe` rule processor on merge to `dev`. Each rule repo holds a caller stub that invokes this reusable definition from `frmscoe/workflows`.

---

## Caller stub (distributed to each rule repo by `sync-workflows.yml`)

```yaml
# .github/workflows/package-rule-rc.yml (in each frmscoe rule repo - managed centrally)
on:
  push:
    branches: [dev]
  workflow_dispatch:
jobs:
  build:
    uses: frmscoe/workflows/.github/workflows/package-rule-rc.yml@dev
    with:
      rule_number: "001"   # zero-padded rule number, e.g. "001", "044"
      rule_org: "frmscoe"
    secrets: inherit
```

---

## Trigger (in each rule repo)

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[dev]` |
| `workflow_dispatch` | manual |

---

## Required Secrets

| Secret | Scope | Purpose |
|--------|-------|---------|
| `GH_TOKEN_LIB` | org | npm install for private packages |
| `DOCKER_USERNAME` | org | Docker Hub authentication |
| `DOCKER_PASSWORD` | org | Docker Hub authentication |
| `SLACK_WEBHOOK_URL` | org | Slack notification |

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | Receives a caller stub |

---

## Known Limitations / Notes

- The comment block in the canonical `frmscoe/workflows/package-rule-rc.yml` file shows a `tazama-lf/workflows` caller stub example - this is incorrect; frmscoe rule repos should reference `frmscoe/workflows`. The caller stubs stamped by `sync-workflows.yml` are correct.
- The checkout step is pinned to `ref: dev`. RC builds must always read the rc line, but `actions/checkout` defaults to the ref that triggered the run - and under `repository_dispatch` (and `workflow_dispatch` once the default branch is `main`) that fallback is the repository default branch, not `dev`. Consumer rule repos pivoted their default branch from `dev` to `main`, so without the explicit `ref: dev` a dispatch-triggered run checked out `main` (a stable version) and failed the prerelease guard. Do not remove the `ref: dev` pin.
- The "Build and push RC Docker image" step's source-branch guard only applies to `push` events. Because the checkout is pinned to `ref: dev`, `repository_dispatch` and `workflow_dispatch` runs always build the dev line even though their `GITHUB_REF` resolves to the default branch (`main`); guarding those events on `GITHUB_REF` would incorrectly refuse the dispatch-triggered rebuilds.
- `dependabot[bot]` actors are excluded.

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | _(caller stubs are identical across all 33 rule repos)_ |

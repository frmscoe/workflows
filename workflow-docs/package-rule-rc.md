# `package-rule-rc.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The reusable workflow logic is identical to the stable variant, but triggers on `push: dev` to produce an `:rc` Docker tag. Caller stubs reference `frmscoe/workflows` and pass `rule_org: "frmscoe"`.
>
> For the full workflow logic documentation and the `tazama-lf` variant, see [`workflow-docs/package-rule-rc.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/package-rule-rc.md).

## Purpose

Reusable workflow that builds and pushes an `:rc` Docker image for a `frmscoe` rule processor on merge to `dev`. Each rule repo holds a caller stub that invokes this reusable definition from `frmscoe/workflows`.

---

## Caller stub (distributed to each rule repo by `sync-workflows.yml`)

```yaml
# .github/workflows/package-rule-rc.yml (in each frmscoe rule repo — managed centrally)
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

- The comment block in the canonical `frmscoe/workflows/package-rule-rc.yml` file shows a `tazama-lf/workflows` caller stub example — this is incorrect; frmscoe rule repos should reference `frmscoe/workflows`. The caller stubs stamped by `sync-workflows.yml` are correct.
- `dependabot[bot]` actors are excluded.

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | _(caller stubs are identical across all 33 rule repos)_ |

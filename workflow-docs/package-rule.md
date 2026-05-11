# `package-rule.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The reusable workflow logic is identical. The difference is that caller stubs distributed to `frmscoe` rule repos reference `frmscoe/workflows` and pass `rule_org: "frmscoe"`.
>
> For the full workflow logic documentation and the `tazama-lf` variant, see [`workflow-docs/package-rule.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/package-rule.md).

## Purpose

Reusable workflow that builds and pushes a stable Docker image for a `frmscoe` rule processor on merge to `main`. Produces both a versioned tag (`:X.Y.Z`) and a `:latest` moving pointer. Each rule repo holds a caller stub that invokes this reusable definition from `frmscoe/workflows`.

---

## Caller stub (distributed to each rule repo by `sync-workflows.yml`)

```yaml
# .github/workflows/package-rule.yml (in each frmscoe rule repo - managed centrally)
on:
  push:
    branches: [main]
  workflow_dispatch:
jobs:
  build:
    uses: frmscoe/workflows/.github/workflows/package-rule.yml@dev
    with:
      rule_number: "001"   # zero-padded rule number, e.g. "001", "044"
      rule_org: "frmscoe"
    secrets: inherit
```

---

## Trigger (in each rule repo)

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[main]` |
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
| All `REPOS` (33 frmscoe rule repos) | Receives a caller stub; the canonical reusable definition stays in `frmscoe/workflows` |

---

## Known Limitations / Notes

- The comment block in the canonical `frmscoe/workflows/package-rule.yml` file shows a `tazama-lf/workflows` caller stub example - this is incorrect; frmscoe rule repos should reference `frmscoe/workflows`. The caller stubs stamped by `sync-workflows.yml` are correct.
- `dependabot[bot]` actors are excluded.

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | _(caller stubs are identical across all 33 rule repos)_ |

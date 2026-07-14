# `package-rule-rc.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The reusable job logic matches the stable variant, but produces RC tags from the source branch (usually `dev`). `frmscoe` rule repos receive caller stubs that reference `frmscoe/workflows` and pass `rule_org: "frmscoe"`.
>
> The **canonical reusable definition stays in this repo**. Sync stamps thin caller stubs into each rule repo; it does not copy this file as-is.

## Purpose

Builds and pushes an **RC** Docker image for a rule processor on the source branch (usually `dev`):

- Versioned tag: `tazamaorg/rule-{NNN}:X.Y.Z-rc.N`
- Moving pointer: `tazamaorg/rule-{NNN}:rc`

Requires a prerelease `package.json` version (fails on clean `X.Y.Z`).

---

## Trigger

### Central reusable workflow

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Invoked by the per-repo caller stub |

### Caller stub (stamped by `sync-workflows.yml`)

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[dev]` |
| `workflow_dispatch` | manual — **must run from the source branch** (`dev`) |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Node version | `22` (overridable) |
| Permissions | `contents: read`, `pull-requests: read` |
| Default Docker namespace | `tazamaorg` |
| Default rule-executer | `tazama-lf/rule-executer@dev` |

---

## Inputs

| Input | Required | Default | Purpose |
|-------|----------|---------|---------|
| `rule_number` | yes | — | Zero-padded rule number, e.g. `"021"` |
| `rule_org` | yes | — | `"frmscoe"` or `"tazama-lf"` |
| `node_version` | no | `22` | Node.js version |
| `rule_executer_repo` | no | `tazama-lf/rule-executer` | Template repository |
| `rule_executer_branch` | no | `dev` | Template branch for RC builds |
| `docker_image_namespace` | no | `tazamaorg` | Docker Hub org/namespace |

### Useful repo / org variables

`NODE_VERSION`, `SOURCE_BRANCH`

---

## Caller stub (distributed to each rule repo by sync)

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
      rule_number: "001"
      rule_org: "frmscoe"
    secrets: inherit
```

---

## Jobs

### `automate-rule-executer`

1. Checkout the rule repository (triggering ref — not force-pinned to `dev`)
2. Set up Node.js
3. Read `package.json` version — fail if missing or **not** a prerelease
4. Clone rule-executer template (`rule_executer_repo` / `rule_executer_branch`, default `dev`)
5. Rewrite rule dependency + Dockerfile env vars for this rule number/org
6. Delete stale lockfile, regenerate with `npm install --package-lock-only`, verify the lock references the correct rule module
7. Branch guard: `GITHUB_REF` must be `refs/heads/${SOURCE_BRANCH||dev}` for **all** events (including `workflow_dispatch`)
8. Build and push `:VERSION` and `:rc`
9. Resolve source PR (best effort) and Slack notify

---

## Required Secrets

| Secret | Scope | Purpose |
|--------|-------|---------|
| `GH_TOKEN_LIB` | org | Private npm / GitHub clone access |
| `DOCKER_USERNAME` | org | Docker Hub login |
| `DOCKER_PASSWORD` | org | Docker Hub login |
| `SLACK_WEBHOOK_URL` | org | Slack notification |

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| All `REPOS` (33 frmscoe rule repos) | Receives a **caller stub**; canonical reusable YAML stays in `frmscoe/workflows` |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e…` | v6.0.2 |
| `actions/setup-node` | `53b83947…` | v6.3.0 |

---

## Known Limitations / Notes

- **No `ref: dev` checkout pin.** Checkout uses the triggering ref. The Docker push step always enforces `SOURCE_BRANCH` (default `dev`).
- Therefore manual `workflow_dispatch` must be started **from the `dev` branch** in the Actions UI. Dispatch from `main` will fail the branch guard (this differs from the previous pin-based behaviour).
- Lockfile regeneration is required so the Docker build does not install a stale rule module from an old lockfile
- Stable counterpart: [`package-rule.yml`](package-rule.md) (`:latest`, target branch)
- `dependabot[bot]` actors are excluded

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | Caller stubs are identical across the 33 rule repos aside from `rule_number` |

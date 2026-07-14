# `package-rule.yml`

> This workflow exists in **both** `tazama-lf/workflows` and `frmscoe/workflows`. The reusable job logic is identical. `frmscoe` rule repos receive caller stubs that reference `frmscoe/workflows` and pass `rule_org: "frmscoe"`.
>
> The **canonical reusable definition stays in this repo**. Sync stamps thin caller stubs into each rule repo; it does not copy this file as-is.

## Purpose

Builds and pushes a **stable** Docker image for a rule processor on merge to the target branch (usually `main`):

- Versioned tag: `tazamaorg/rule-{NNN}:X.Y.Z`
- Moving pointer: `tazamaorg/rule-{NNN}:latest`

Rejects prerelease `package.json` versions (companion: [`version-check.yml`](version-check.md)).

---

## Trigger

### Central reusable workflow

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Invoked by the per-repo caller stub |

### Caller stub (stamped by `sync-workflows.yml`)

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[main]` |
| `workflow_dispatch` | manual — must run from the target branch |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Node version | `22` (overridable) |
| Permissions | `contents: read`, `pull-requests: read` |
| Default Docker namespace | `tazamaorg` |
| Default rule-executer | `tazama-lf/rule-executer@main` |

---

## Inputs

| Input | Required | Default | Purpose |
|-------|----------|---------|---------|
| `rule_number` | yes | — | Zero-padded rule number, e.g. `"021"` |
| `rule_org` | yes | — | `"frmscoe"` or `"tazama-lf"` |
| `node_version` | no | `22` | Node.js version |
| `rule_executer_repo` | no | `tazama-lf/rule-executer` | Template repository |
| `rule_executer_branch` | no | `main` | Template branch for stable builds |
| `docker_image_namespace` | no | `tazamaorg` | Docker Hub org/namespace |

### Useful repo / org variables

`NODE_VERSION`, `TARGET_BRANCH`

---

## Caller stub (distributed to each rule repo by sync)

```yaml
# .github/workflows/package-rule.yml (in each frmscoe rule repo - managed centrally)
on:
  push:
    branches: [main]
  workflow_dispatch:
jobs:
  build:
    uses: frmscoe/workflows/.github/workflows/package-rule.yml@main
    with:
      rule_number: "001"
      rule_org: "frmscoe"
    secrets: inherit
```

> Sync stamps `@main` for the stable caller. Do not point stable builds at `@dev` unless intentionally testing.

---

## Jobs

### `automate-rule-executer`

1. Checkout the rule repository (triggering ref)
2. Set up Node.js
3. Read `package.json` version — fail if missing or prerelease
4. Clone rule-executer template (`rule_executer_repo` / `rule_executer_branch`)
5. Rewrite rule dependency + Dockerfile `RULE_NAME` / `APM_SERVICE_NAME` for this rule number/org
6. `npm install` in the prepared executer tree
7. Branch guard: push only from `TARGET_BRANCH` (default `main`)
8. Build and push `:VERSION` and `:latest`
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

- Header comments in the YAML may still show a `tazama-lf` caller example — stamped frmscoe stubs are authoritative
- Namespace/org rewrite validates after sed; template pattern changes will fail fast
- Image namespace and rule-executer repo/branch are now inputs (no longer fully hard-coded)
- `dependabot[bot]` actors are excluded

---

## Repository Overrides

| Repository | Reason |
|-----------|--------|
| _(none)_ | Caller stubs are identical across the 33 rule repos aside from `rule_number` |

<!-- SPDX-License-Identifier: Apache-2.0 -->

# frmscoe/workflows

> **⚠️ This repository is a downstream mirror of [`tazama-lf/workflows`](https://github.com/tazama-lf/workflows), which is the canonical source for all Tazama GitHub Actions workflows.**
>
> **All workflow changes must originate in `tazama-lf/workflows`.** There is no automated sync between the two workflow repos — changes must be applied here manually after merging in `tazama-lf/workflows`. **Do not edit workflow files in this repo directly** without a corresponding change upstream.

For complete SDLC documentation, repository class definitions, workflow reference tables, routine maintenance procedures, and known issues, see the **[`tazama-lf/workflows` README](https://github.com/tazama-lf/workflows/blob/dev/README.md)**.

---

## What this repo is

This repository holds the GitHub Actions workflows distributed to all active `frmscoe` organisation repositories (33 rule repos). It is a manually-maintained subset of `tazama-lf/workflows`, adapted for the `frmscoe` org context.

---

## Cascade and dependency

Changes flow in one direction:

```
tazama-lf/workflows  →  (manual PR)  →  frmscoe/workflows  →  (auto sync on push:dev)  →  33 frmscoe rule repos
```

1. A workflow change is developed and merged to `dev` in `tazama-lf/workflows`.
2. The same change is applied manually to `frmscoe/workflows` via a separate PR.
3. On merge to `dev` in `frmscoe/workflows`, `sync-workflows.yml` fires automatically (`push: dev`) and opens `sync-workflows-update` PRs in all 33 target rule repos.
4. Reviewers merge the sync PRs in each rule repo.

> **Note:** Unlike `tazama-lf/workflows` (which triggers sync on every PR event before merge), `frmscoe/workflows` triggers sync only on `push: dev` — i.e. after merge. Sync PRs in rule repos accurately reflect merged changes.

---

## Differences from tazama-lf/workflows

| Aspect | `frmscoe/workflows` | `tazama-lf/workflows` |
|--------|--------------------|-----------------------|
| npm scope | `@frmscoe` | `@tazama-lf` |
| Docker caller stub org | `rule_org: "frmscoe"` | `rule_org: "tazama-lf"` |
| Sync trigger | `push: dev` (after merge) | `pull_request: [dev]` (on open/update) |
| Sync targets | 33 frmscoe rule repos | 26 tazama-lf repos |
| Sync segmentation | None — all repos receive the same file set | `SPECIFIC_REPOS` / `PUBLISH_REPOS` / `RULE_REPOS` groups |
| Missing workflows | `dockerfile-linter.yml`, `dockerhub-image-build.yml`, `dockerhub-image-build-rc.yml` | All canonical files present |
| Node.js CI | `node-ci.yml` uses `NPM_SCOPE: @frmscoe`; stub calls `frmscoe/workflows/node-ci.yml@dev` | `node-ci.yml` uses `NPM_SCOPE: @tazama-lf`; stub calls `tazama-lf/workflows/node-ci.yml@dev` |

---

## Workflows distributed to frmscoe rule repos

All 33 rule repos receive:

`branch-target-check.yml`, `codacy.yml`, `codeql.yml`, `conventional-commits.yml`, `dco-check.yml`, `dependency-review.yml`, `gpg-verify.yml`, `milestone.yml`, `njsscan.yml`, `node.js.yml` (caller stub), `publish.yml`, `release-train.yml`, `release.yml`, `sbom.yml`, `scorecard.yml`, `version-check.yml`

Plus per-repo caller stubs for: `package-rule-rc.yml` (fires on `push: dev`) and `package-rule.yml` (fires on `push: main`)

**Not distributed:** `sync-workflows.yml`, `node-ci.yml` (reusable workflow stays in this repo; consumer repos reference it at runtime via `@dev` ref), `package-rule*.yml` canonical reusable definitions (replaced with caller stubs)

---

## Target repositories

`rule-001`, `rule-002`, `rule-003`, `rule-004`, `rule-006`, `rule-007`, `rule-008`, `rule-010`, `rule-011`, `rule-016`, `rule-017`, `rule-018`, `rule-020`, `rule-021`, `rule-024`, `rule-025`, `rule-026`, `rule-027`, `rule-028`, `rule-030`, `rule-044`, `rule-045`, `rule-048`, `rule-054`, `rule-063`, `rule-074`, `rule-075`, `rule-076`, `rule-078`, `rule-083`, `rule-084`, `rule-090`, `rule-091`

---

## Workflow documentation

Individual workflow documentation is in [`workflow-docs/`](workflow-docs/). For workflows shared with `tazama-lf/workflows`, docs contain a redirect link to the canonical entry in that repo. Docs for frmscoe-specific behaviour (`publish.yml`, `package-rule*.yml`, `sync-workflows.yml`) are maintained here.

| Doc | Status |
|-----|--------|
| [`branch-target-check.md`](workflow-docs/branch-target-check.md) | → tazama-lf docs |
| [`codacy.md`](workflow-docs/codacy.md) | → tazama-lf docs |
| [`codeql.md`](workflow-docs/codeql.md) | → tazama-lf docs |
| [`conventional-commits.md`](workflow-docs/conventional-commits.md) | → tazama-lf docs |
| [`dco-check.md`](workflow-docs/dco-check.md) | → tazama-lf docs (⚠️ known issue [#37](https://github.com/tazama-lf/workflows/issues/37)) |
| [`dependency-review.md`](workflow-docs/dependency-review.md) | → tazama-lf docs |
| [`dockerfile-linter.md`](workflow-docs/dockerfile-linter.md) | Not in frmscoe/workflows |
| [`dockerhub-image-build.md`](workflow-docs/dockerhub-image-build.md) | Not in frmscoe/workflows |
| [`gpg-verify.md`](workflow-docs/gpg-verify.md) | → tazama-lf docs |
| [`milestone.md`](workflow-docs/milestone.md) | → tazama-lf docs |
| [`njsscan.md`](workflow-docs/njsscan.md) | → tazama-lf docs |
| [`node-ci.md`](workflow-docs/node-ci.md) | frmscoe-specific (`NPM_SCOPE=@frmscoe`) |
| [`nodejs.md`](workflow-docs/nodejs.md) | → tazama-lf docs |
| [`package-rule-rc.md`](workflow-docs/package-rule-rc.md) | frmscoe-specific |
| [`package-rule.md`](workflow-docs/package-rule.md) | frmscoe-specific |
| [`publish.md`](workflow-docs/publish.md) | frmscoe-specific (`@frmscoe` scope) |
| [`release-train.md`](workflow-docs/release-train.md) | → tazama-lf docs |
| [`release.md`](workflow-docs/release.md) | → tazama-lf docs |
| [`sbom.md`](workflow-docs/sbom.md) | → tazama-lf docs (⚠️ known issue [#39](https://github.com/tazama-lf/workflows/issues/39)) |
| [`scorecard.md`](workflow-docs/scorecard.md) | → tazama-lf docs |
| [`sync-workflows.md`](workflow-docs/sync-workflows.md) | frmscoe-specific |
| [`version-check.md`](workflow-docs/version-check.md) | → tazama-lf docs |

---

## Updating this repo

To apply a workflow change from `tazama-lf/workflows`:

1. Confirm the source PR in `tazama-lf/workflows` is merged to `dev`.
2. Open a PR in this repo with the same changes, referencing the source PR.
3. Update the corresponding `workflow-docs/` entry if the frmscoe behaviour differs.
4. On merge to `dev`, `sync-workflows.yml` will open `sync-workflows-update` PRs in all 33 target repos.
5. Review and merge the sync PRs in each target repo.

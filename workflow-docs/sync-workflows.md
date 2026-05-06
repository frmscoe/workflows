# `sync-workflows.yml`

> The `sync-workflows.yml` in `frmscoe/workflows` has **different trigger behaviour and a different target set** from the canonical version in `tazama-lf/workflows`. This document covers the `frmscoe/workflows` variant.
>
> For the `tazama-lf/workflows` variant, see [`workflow-docs/sync-workflows.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/sync-workflows.md).

## Purpose

Propagates canonical workflow files and the standard `.codacy.yml` engine allowlist from this repository to all 33 active `frmscoe` rule repos. Fires automatically on `push: dev` (i.e. after a PR is merged). All rule repos receive the same file set - there is no segmentation by repo type. Caller stubs for `package-rule*.yml` are stamped individually per repo with the correct rule number.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[dev]` — fires after merge, not on PR open |
| `workflow_dispatch` | manual |

> **Key difference from `tazama-lf/workflows`:** This variant triggers on `push: dev` (fires once, after merge). The `tazama-lf` variant triggers on all `pull_request` events to `dev` (fires on open and update, before merge). See [tazama-lf/workflows#36](https://github.com/tazama-lf/workflows/issues/36).

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Target org | `frmscoe` |
| Target repos | 33 rule repos (active subset of `rule-001`–`rule-091`) |
| Segmentation | None — all repos receive the same file set |
| Typical duration | ~20–40 min |
| Permissions | default (plus `GH_TOKEN` for cross-repo operations) |

---

## Target Repos

`rule-001`, `rule-002`, `rule-003`, `rule-004`, `rule-006`, `rule-007`, `rule-008`, `rule-010`, `rule-011`, `rule-016`, `rule-017`, `rule-018`, `rule-020`, `rule-021`, `rule-024`, `rule-025`, `rule-026`, `rule-027`, `rule-028`, `rule-030`, `rule-044`, `rule-045`, `rule-048`, `rule-054`, `rule-063`, `rule-074`, `rule-075`, `rule-076`, `rule-078`, `rule-083`, `rule-084`, `rule-090`, `rule-091`

---

## Jobs

### `Sync_All_Repos_Common_Workflows`

**Steps:**

1. `actions/checkout@v4` — checks out this repo
2. `Set up Git` — configures git identity for commits
3. `Install GitHub CLI` — downloads and installs `gh` CLI v2.14.7
4. `Get PR author details` — captures author name and email for commit attribution
5. `Sync Workflows to Other Repos` — main loop:
   - Clones each rule repo from `https://github.com/frmscoe/<repo>`
   - Checks out or creates the `sync-workflows-update` branch
   - Copies all files from the bundle **except** `package-rule*.yml` canonical definitions
   - Stamps a `package-rule-rc.yml` caller stub (referencing `frmscoe/workflows`, `push: [dev]`)
   - Stamps a `package-rule.yml` caller stub (referencing `frmscoe/workflows`, `push: [main]`)
   - Both stubs pass `rule_org: "frmscoe"` and the repo's zero-padded rule number
   - Copies `config-templates/.codacy.yml` to the repo root as `.codacy.yml` (standard Codacy engine allowlist for TypeScript/Node.js repos)
   - Commits with `[skip ci]` in the commit message to suppress CI on the sync commit itself
   - Pushes, opens `sync-workflows-update` PR targeting `dev` with `[skip ci]` in the PR title (so squash-merging the PR also skips CI)

---

## Workflows excluded from the sync bundle

| File | Reason |
|------|--------|
| `sync-workflows.yml` | Canonical-only; never distributed |
| `node-ci.yml` | Reusable workflow; consumer repos reference it at runtime via `@dev` |
| `package-rule*.yml` (canonical definitions) | Replaced with per-repo caller stubs (see above) |

---

## Required Secrets

| Secret | Scope | Purpose |
|--------|-------|-------|
| `GH_TOKEN` | org | Clone, push, and open PRs in target repos |
| `PR_REVIEWERS` | vars | Reviewer login(s) for sync PRs |

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| **Not synced** | This file is excluded from the sync bundle |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|----------|
| `actions/checkout` | tag ref `v4` | — |

---

## Known Limitations / Notes

- `gh` CLI is pinned to v2.14.7 via a hardcoded tarball URL; update the download URL and extracted paths in the `Install GitHub CLI` step when upgrading.
- This repo is a manually-maintained mirror of `tazama-lf/workflows`. Changes to shared workflow files must originate in `tazama-lf/workflows` and be applied here separately - there is no automated sync between the two workflow repos.
- **`[skip ci]` in sync commits and PR titles:** Both the commit message and the PR title include `[skip ci]`. This suppresses CI on the sync commit itself (avoiding unnecessary workflow runs triggered by the push to `sync-workflows-update`). When a sync PR is squash-merged, GitHub uses the PR title as the squash commit message, so `[skip ci]` propagates to the merge commit automatically.
- `dependabot[bot]` actors are excluded.

---

## Repository Overrides

Not applicable — this workflow is canonical-only and is never distributed.

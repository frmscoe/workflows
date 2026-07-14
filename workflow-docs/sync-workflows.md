# `sync-workflows.yml`

> The `sync-workflows.yml` in `frmscoe/workflows` has **different trigger behaviour and a different target set** from the canonical version in `tazama-lf/workflows`. This document covers the `frmscoe/workflows` variant.
>
> For the `tazama-lf/workflows` variant, see [`workflow-docs/sync-workflows.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/sync-workflows.md).

## Purpose

Propagates **day-to-day CI** workflow files and the standard `.codacy.yml` from this repository to all 33 active `frmscoe` rule repos. Fires automatically on `push: dev` (after a PR is merged).

This is **not** the installer for platform release callers. Release/reusable workflows (`release-train`, `publish`, `release`, `dev-to-main-pr`, and the full `package-rule*` definitions) stay central-only; rule repos get thin `package-rule*` caller stubs stamped by this workflow.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[dev]` — fires after merge, not on PR open |
| `workflow_dispatch` | manual |

> **Key difference from `tazama-lf/workflows`:** This variant triggers on `push: dev` (once, after merge). The `tazama-lf` variant triggers on `pull_request` events to `dev` (before merge). See [tazama-lf/workflows#36](https://github.com/tazama-lf/workflows/issues/36).

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Target org | `frmscoe` |
| Target repos | 33 rule repos (active subset of `rule-001`–`rule-091`) |
| Segmentation | None — all repos receive the same filtered file set |
| Concurrency | `sync-workflows-${{ github.ref }}` (cancel-in-progress) |
| Typical duration | ~20–40 min |

---

## Target Repos

`rule-001`, `rule-002`, `rule-003`, `rule-004`, `rule-006`, `rule-007`, `rule-008`, `rule-010`, `rule-011`, `rule-016`, `rule-017`, `rule-018`, `rule-020`, `rule-021`, `rule-024`, `rule-025`, `rule-026`, `rule-027`, `rule-028`, `rule-030`, `rule-044`, `rule-045`, `rule-048`, `rule-054`, `rule-063`, `rule-074`, `rule-075`, `rule-076`, `rule-078`, `rule-083`, `rule-084`, `rule-090`, `rule-091`

---

## Jobs

### `Sync_All_Repos_Common_Workflows`

1. Checkout this workflows repo
2. Configure git identity and **SSH commit signing** (`SSH_SIGNING_KEY` required)
3. Capture actor details for DCO `Signed-off-by`
4. Build a filtered temp workflow bundle, then for each rule repo:
   - Clone the repo and ensure `dev` exists
   - Recreate reserved branch `sync-workflows-update`
   - Copy remaining workflow files from the filtered bundle
   - Stamp `package-rule-rc.yml` caller (`uses: …@dev`, `push: [dev]`)
   - Stamp `package-rule.yml` caller (`uses: …@main`, `push: [main]`)
   - Copy `config-templates/.codacy.yml` → `.codacy.yml`
   - Remove legacy `dco-check.yaml` if present
   - Commit with `[skip ci]`, push, open/update PR to `dev` with `[skip ci]` in the title

Uses the runner-provided `gh` CLI (no separate install step).

---

## Workflows excluded from the sync bundle

| File | Reason |
|------|--------|
| `sync-workflows.yml` | Canonical-only; never distributed |
| `*-ci.yml` / reusable CI implementations | Stay in this repo; consumers call them at runtime via `@dev` |
| `package-rule.yml` / `package-rule-rc.yml` (canonical) | Replaced with per-repo caller stubs |
| `publish.yml` | Release reusable — bootstrap callers for package repos only |
| `release.yml` | Release reusable — bootstrap callers only |
| `release-train.yml` | Release reusable — bootstrap callers only |
| `dev-to-main-pr.yml` | Release reusable — bootstrap callers for non-code repos only |

`version-check.yml` remains in the sync bundle so rule repos get the PR-to-`main` prerelease gate that pairs with stable `package-rule.yml`.

---

## Required Secrets

| Secret | Scope | Purpose |
|--------|-------|---------|
| `GH_TOKEN` | org | Clone, push, and open PRs in target repos |
| `GH_USERNAME` | org | Reviewer login(s) for sync PRs (`PR_REVIEWERS`) |
| `SSH_SIGNING_KEY` | org | Base64-encoded SSH private key for verified sync commits |

`SSH_SIGNING_KEY` must be set with `base64 -w 0 signing_key | gh secret set SSH_SIGNING_KEY`. The matching public key must be registered as a **Signing Key** on the GitHub account that owns `GH_TOKEN`.

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| **Not synced** | This file itself is excluded from the sync bundle |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e…` | v6.0.2 |

---

## Known Limitations / Notes

- This repo is a manually-maintained mirror of `tazama-lf/workflows`. Shared workflow changes must originate upstream and be applied here separately
- **`[skip ci]`** in sync commits and PR titles suppresses CI on the sync push and squash-merge
- Release callers for libraries / non-code repos are installed by **bootstrap scripts**, not this sync
- Reserved branch name: `sync-workflows-update` — do not use for normal development
- `dependabot[bot]` actors are excluded

---

## Repository Overrides

Not applicable — this workflow is canonical-only and is never distributed.

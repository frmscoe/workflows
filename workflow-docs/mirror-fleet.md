# `mirror-fleet.yml`

> Canonical-only workflow in `frmscoe/workflows`. It has no equivalent in `tazama-lf/workflows` and is never distributed to the rule repos.

## Purpose

Pushes the released `main` branch and all tags of each private `frmscoe` repo to its read-only mirror in the [`tazama-rules`](https://github.com/tazama-rules) org. The mirrors give community members free read access to the rule processor source and the full configuration without consuming GitHub Enterprise seats in `frmscoe` (TSC decision: [tazama-lf/technical-steering-committee#38](https://github.com/tazama-lf/technical-steering-committee/issues/38)).

This is a **deliberate release-day step**: the workflow has no automatic triggers. Mirrors only change when a maintainer dispatches this workflow, which should happen once per release after `main` is final on the canonical repos.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `workflow_dispatch` | manual only - no `push`, no schedule |

### Inputs

| Input | Required | Default | Purpose |
|-------|----------|---------|---------|
| `repos` | no | `''` | Comma-separated subset to mirror (e.g. `tms-configuration` or `rule-001,rule-002`). Empty = full fleet. |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Source org | `frmscoe` (canonical private repos) |
| Target org | `tazama-rules` (read-only mirrors) |
| Target repos | 34: the 33 active rule repos + `tms-configuration` |
| Permissions | `permissions: {}` - the job needs no access to this repo; all access is via secrets |

---

## Target Repos

The fleet is defined in the `FLEET` env var in the workflow file:

`rule-001`, `rule-002`, `rule-003`, `rule-004`, `rule-006`, `rule-007`, `rule-008`, `rule-010`, `rule-011`, `rule-016`, `rule-017`, `rule-018`, `rule-020`, `rule-021`, `rule-024`, `rule-025`, `rule-026`, `rule-027`, `rule-028`, `rule-030`, `rule-044`, `rule-045`, `rule-048`, `rule-054`, `rule-063`, `rule-074`, `rule-075`, `rule-076`, `rule-078`, `rule-083`, `rule-084`, `rule-090`, `rule-091`, `tms-configuration`

This list must be kept in step with the active-repo inventory (`release-repo-list.json` `frmscoe` array). When a rule repo is added or retired: update `FLEET` here, and create or archive the matching mirror repo in `tazama-rules`.

---

## Jobs

### `mirror`

**Steps:**

1. `Mint mirror push token` - `actions/create-github-app-token` mints a short-lived installation token for the `tazama-rules-mirror-sync` GitHub App (installed org-wide on `tazama-rules` with `contents: write`).
2. `Mirror main + tags` - main loop over the repo list:
   - Clones the canonical repo's `main` (single-branch) from `frmscoe` using `GH_TOKEN`
   - Fetches all tags
   - Force-pushes `main` and `refs/tags/*` to `tazama-rules/<repo>` using the App token
   - A per-repo failure is tolerated: the loop continues, the repo is recorded as **FAILED** in the step summary, and the job exits `1` at the end so the run is marked failed

The run's step summary contains a per-repo result table - check it after every dispatch.

**What the mirrors receive:** an exact copy of `main` plus tags, including original commit signatures and verification badges. **Not mirrored:** other branches, issues, PRs, releases (the release *tags* carry over; GitHub Release objects do not), Actions workflows runs.

---

## Required Secrets

| Secret | Scope | Purpose |
|--------|-------|-------|
| `GH_TOKEN` | repo (`frmscoe/workflows`) | Read access to clone the canonical private repos (same PAT used by `sync-workflows.yml`) |
| `MIRROR_APP_ID` | repo (`frmscoe/workflows`) | App ID of the `tazama-rules-mirror-sync` GitHub App |
| `MIRROR_APP_PRIVATE_KEY` | repo (`frmscoe/workflows`) | Private key (PEM) of the mirror-sync App |

---

## Sync Distribution

| Group | Behaviour |
|-------|-----------|
| **Not synced** | Canonical-only; `sync-workflows.yml` explicitly removes `mirror-fleet.yml` from the fan-out bundle |

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|----------|
| `actions/create-github-app-token` | `bcd2ba49218906704ab6c1aa796996da409d3eb1` | v3.2.0 |

---

## Known Limitations / Notes

- **Force-push by design.** Mirrors are verbatim copies; any local change on a mirror (including a hand-edited README) is wiped on the next sync. The mirror repos carry their pointer-to-canonical in the repo *description* instead.
- **Dispatch from `dev`.** The workflow lives on `dev` (this repo's default working branch): `gh workflow run mirror-fleet.yml -R frmscoe/workflows --ref dev`.
- **Tags are force-updated** (`refs/tags/*:refs/tags/*` under `--force`), so a re-tagged release on the canonical side propagates.
- **New mirror repos are not auto-created.** The mirror push fails for a repo that has no counterpart in `tazama-rules`; create it first (private, issues/wiki/projects disabled, description pointing at the canonical repo).

---

## Repository Overrides

Not applicable - this workflow is canonical-only and is never distributed.

---

## Release-day checklist

Run this after `main` is final on all released repos:

1. Dispatch the full fleet: `gh workflow run mirror-fleet.yml -R frmscoe/workflows --ref dev` (empty `repos` input = all 34 repos).
2. Open the run's step summary and check the per-repo result table. Re-run any failed repo individually: `gh workflow run mirror-fleet.yml -R frmscoe/workflows --ref dev -f repos=<name>`.
3. Spot-check one mirror against its canonical:
   - `gh api repos/tazama-rules/tms-configuration/commits/main --jq '.sha'` matches `gh api repos/frmscoe/tms-configuration/commits/main --jq '.sha'`
   - The release tag is present on the mirror: `gh api repos/tazama-rules/tms-configuration/tags --jq '.[].name'`
4. If a new rule repo joined the fleet this release: confirm its mirror repo exists in `tazama-rules` and its name is in `FLEET`.

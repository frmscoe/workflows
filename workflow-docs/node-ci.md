# `node-ci.yml`

> **frmscoe variant** of the reusable Node.js CI workflow. The upstream canonical version lives in [`tazama-lf/workflows`](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/node-ci.md). The only difference is the `NPM_SCOPE` value (`@frmscoe` vs `@tazama-lf`).

## Purpose

Reusable Node.js CI pipeline with three parallel jobs - build, lint, and test - running against Node.js 20. Contains all CI logic for Node.js consumer repos. Called by the [`node.js.yml`](nodejs.md) caller stub which is synced to all 33 rule repos.

This file is **not synced** to consumer repos - it stays in `frmscoe/workflows` and is referenced by the stub at runtime via `uses: frmscoe/workflows/.github/workflows/node-ci.yml@dev`.

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `workflow_call` | Called by `node.js.yml` stub in consumer repos |

---

## Execution Context

| Property | Value |
|----------|-------|
| Runner | `ubuntu-latest` |
| Node version | `20` |
| Permissions | `contents: read` |

---

## Environment Variables

| Variable | Value | Purpose |
|----------|-------|---------|
| `NPM_SCOPE` | `@frmscoe` | npm scope for GitHub Packages registry routing |
| `NPM_REGISTRY` | `https://npm.pkg.github.com/` | GitHub Packages registry URL |
| `NODE_ENV` | `test` | sets test environment for all jobs |

---

## Jobs

### `build` - run build

1. `actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd` (v6.0.2)
2. `actions/setup-node@53b83947a5a98c8d113130e565377fae1a50d02f` (v6.3.0) - Node 20, npm cache, registry and scope
3. `npm ci` (authenticated via `NODE_AUTH_TOKEN: secrets.GITHUB_TOKEN`)
4. `npm run build`

### `lint` - check style

1. `actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd` (v6.0.2)
2. `actions/setup-node@53b83947a5a98c8d113130e565377fae1a50d02f` (v6.3.0)
3. `npm ci` (authenticated via `NODE_AUTH_TOKEN: secrets.GITHUB_TOKEN`)
4. `npm run lint`

### `test` - check tests

1. `actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd` (v6.0.2)
2. `actions/setup-node@53b83947a5a98c8d113130e565377fae1a50d02f` (v6.3.0)
3. `npm ci` (authenticated via `NODE_AUTH_TOKEN: secrets.GITHUB_TOKEN`)
4. `npm test`

---

## Sync Distribution

| Group | Behaviour |
|-------|----------|
| **All repos** | **Not synced** - reusable workflow stays in `frmscoe/workflows` only; consumer repos reference it by the `@dev` branch ref at runtime |

The caller stub (`node.js.yml`) is what gets synced. See [`nodejs.md`](nodejs.md).

---

## Dependencies (pinned actions)

| Action | Pinned SHA | Semver alias |
|--------|-----------|--------------|
| `actions/checkout` | `de0fac2e4500dabe0009e67214ff5f5447ce83dd` | v6.0.2 |
| `actions/setup-node` | `53b83947a5a98c8d113130e565377fae1a50d02f` | v6.3.0 |

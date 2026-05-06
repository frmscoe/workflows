# `node.js.yml`

Caller stub that delegates Node.js CI to the centralised reusable workflow [`node-ci.yml`](node-ci.md). Contains only the push/PR triggers and a single `uses:` reference - no logic lives here. This file is synced to all 33 rule repos unchanged.

> **Upstream reference:** [`workflow-docs/nodejs.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/nodejs.md) - the architecture is identical; only the reusable workflow org differs (`frmscoe` vs `tazama-lf`).

---

## Trigger

| Event | Conditions |
|-------|-----------|
| `push` | branches: `[dev, main]` |
| `pull_request` | branches: `[dev, main]` |

---

## Jobs

### `node-ci`

Delegates entirely to the reusable workflow:

```yaml
uses: frmscoe/workflows/.github/workflows/node-ci.yml@dev
secrets: inherit
```

See [`node-ci.yml` documentation](node-ci.md) for the full job breakdown.

---

## Permissions

| Scope | Level |
|-------|-------|
| `contents` | `read` |

---

## Sync Distribution

Synced to all 33 frmscoe rule repos unchanged.

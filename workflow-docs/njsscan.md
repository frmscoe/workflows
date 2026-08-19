# `njsscan.yml`

> This workflow is distributed from [`tazama-lf/workflows`](https://github.com/tazama-lf/workflows) without modification. For full documentation - including trigger details, job steps, required secrets, and known limitations - see:
>
> **[`workflow-docs/njsscan.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/njsscan.md)**

## frmscoe-specific notes

**Not distributed to private `frmscoe` rule repos.** SARIF upload needs GitHub Code Security on private repos (billable). Decision: keep Node security scanning on the public reference rules [`tazama-lf/rule-901`](https://github.com/tazama-lf/rule-901) and [`tazama-lf/rule-902`](https://github.com/tazama-lf/rule-902), and propagate fixes through the normal central-workflow path. See the README section *Decision: no dedicated Code Security scanning on private frmscoe rule repos*.

The caller stub and `njsscan-ci.yml` remain in this repo for reference. `sync-workflows.yml` excludes them from the private-rule bundle and removes leftover copies on sync.

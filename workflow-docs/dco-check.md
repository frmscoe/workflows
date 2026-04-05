# `dco-check.yml`

> This workflow is distributed from [`tazama-lf/workflows`](https://github.com/tazama-lf/workflows) without modification. For full documentation — including trigger details, job steps, required secrets, and known limitations — see:
>
> **[`workflow-docs/dco-check.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/dco-check.md)**

## frmscoe-specific notes

⚠️ The `git log` range in this workflow is reversed — it checks commits in the base branch that are not in the head branch, rather than the PR's new commits. DCO sign-off is not currently being verified correctly. This is a known issue tracked in [tazama-lf/workflows#37](https://github.com/tazama-lf/workflows/issues/37).

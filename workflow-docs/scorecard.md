# `scorecard.yml`

> This workflow is distributed from [`tazama-lf/workflows`](https://github.com/tazama-lf/workflows) without modification. For full documentation — including trigger details, job steps, required secrets, and known limitations — see:
>
> **[`workflow-docs/scorecard.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/scorecard.md)**

## frmscoe-specific notes

In `frmscoe/workflows`, `scorecard.yml` is distributed to all 33 rule repos (there is no `PUBLISH_REPOS` exclusion in the frmscoe sync). In `tazama-lf/workflows`, `scorecard.yml` is excluded from `PUBLISH_REPOS`; frmscoe has no equivalent segmentation.

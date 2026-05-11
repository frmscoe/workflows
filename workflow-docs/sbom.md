# `sbom.yml`

> This workflow is distributed from [`tazama-lf/workflows`](https://github.com/tazama-lf/workflows) without modification. For full documentation - including trigger details, job steps, required secrets, and known limitations - see:
>
> **[`workflow-docs/sbom.md` in tazama-lf/workflows](https://github.com/tazama-lf/workflows/blob/dev/workflow-docs/sbom.md)**

## frmscoe-specific notes

⚠️ `sbom.yml` runs `docker build` to produce the SBOM. `frmscoe` rule repos do not contain a `Dockerfile`, so this workflow fails on every `push: main` in rule repos. This is a known issue tracked in [tazama-lf/workflows#39](https://github.com/tazama-lf/workflows/issues/39).

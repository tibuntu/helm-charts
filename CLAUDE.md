# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common commands

```bash
# Lint a chart
helm lint charts/<name>

# Render templates locally for inspection
helm template <release-name> charts/<name>

# Render with custom values
helm template <release-name> charts/<name> -f my-values.yaml

# Regenerate chart READMEs (requires helm-docs)
helm-docs --chart-search-root=charts

# Run pre-commit hooks locally
pre-commit run --all-files
```

## Release process

Releases are fully automated via `helm/chart-releaser-action` (`.github/workflows/helm.yml`). To publish a new chart version:

1. Bump `version` in the chart's `Chart.yaml`
2. Update the `artifacthub.io/changes` annotation in `Chart.yaml`
3. Merge to `main` — the release workflow creates a GitHub Release tagged `<chart-name>-<version>`, packages the chart tarball, and updates `index.yaml` on the `gh-pages` branch

The release workflow only triggers on pushes that modify a `Chart.yaml` file, so routine changes (templates, values, docs) do not attempt a release.

## Automation

- **`docs.yml`** — runs on pushes to `main` that touch `Chart.yaml`, `values.yaml`, or `*.gotmpl` files. Runs `helm-docs` to update per-chart `README.md` files and regenerates the `## Charts` list in the root `README.md` from all `charts/*/Chart.yaml` entries. Auto-commits with `[skip ci]`.
- **`lint.yml`** — runs `helm lint` on all charts on every PR and main push.
- **Renovate** — updates GitHub Actions versions (`config:recommended`) and pre-commit hook versions (`pre-commit` manager enabled).

## Chart conventions

Both charts share a common structure and set of conventions:

- **Security defaults**: containers run as UID/GID 911, `readOnlyRootFilesystem: true`, `runAsNonRoot: true`, all Linux capabilities dropped.
- **Image**: the `generic` chart constructs the image as `registry/repository:tag`. Both `registry` and `repository` must be set.
- **`environment`**: takes a YAML list (despite the `{}` default), passed directly to `env:` — use the standard Kubernetes `name`/`value` format.
- **Probes**: disabled by default in `generic` (toggle via `livenessProbe.enabled` / `readinessProbe.enabled`); hardcoded in `voyager`.
- **Persistence** (`generic` only): when `persistence.enabled: true`, a PVC is created and mounted. The PVC name defaults to `<release-name>-pvc`.
- **Resources**: empty by default in `generic`; explicit requests/limits in `voyager`.
- **Ingress**: templates support both `networking.k8s.io/v1` (k8s ≥1.19) and `extensions/v1beta1` (older) via version detection.

## Adding a new chart

1. Create `charts/<name>/` with `Chart.yaml`, `values.yaml`, `templates/`, and `templates/_helpers.tpl`
2. Set `home: https://github.com/tibuntu/helm-charts/tree/main/charts/<name>` in `Chart.yaml`
3. The root `README.md` chart list updates automatically on next push via `docs.yml`

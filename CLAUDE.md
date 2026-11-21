# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Only the `generic` chart remains. Voyager runs through it (values snippet in the root `README.md`, "Running Voyager").

## Common commands

```bash
helm lint --strict charts/generic
helm template <release-name> charts/generic -f charts/generic/ci/full-values.yaml
helm-docs --chart-search-root=charts   # regenerate chart READMEs
pre-commit run --all-files
```

## Release process

Releases are automated by `.github/workflows/helm.yml` (`helm/chart-releaser-action`). It runs only on pushes to `main` that touch a `Chart.yaml`, and calls `ct.yml` as job `test`; `release` needs it. A version-bump commit therefore runs chart testing twice (accepted).

1. Bump `version` in `Chart.yaml`. The chart is 0.x: a breaking change bumps the minor version.
2. Replace the `artifacthub.io/changes` annotation with this version's entries only (it describes the current version, not history). A `kind: changed` entry says how to migrate. `helm.yml` renders these entries into `RELEASE-NOTES.md`, which `cr.yaml` makes the GitHub Release body (Renovate shows it in update PRs).
3. Merge to `main`. A GitHub Release `<chart-name>-<version>` is created, `index.yaml` on `gh-pages` updated, and the package pushed to `oci://ghcr.io/tibuntu/charts` (existing versions are skipped, never overwritten).

`ct lint` fails a PR that changes anything under `charts/<name>/` (including `ci/`) without a version bump. That includes Renovate PRs that only bump an image in `ci/` or `templates/tests/`: bump the version by hand.

## Automation

- **`ct.yml`** (chart testing, `ct.yaml`): on PRs it lints and installs (kind) only changed charts; on push to `main` and `workflow_dispatch` it tests all charts without the version-increment check. Lint runs with every `ci/*-values.yaml`. `ct` leaves untracked `charts/*/Chart.lock`; do not commit it.
- **`pre-commit.yml`** runs `pre-commit run --all-files` on PRs and main pushes with `SKIP=helm-docs` (`docs.yml` regenerates READMEs). `ct.yaml` points `lint-conf` at `.yamllint`, so ct and pre-commit share yamllint rules.
- **`docs.yml`** on pushes to `main` touching `Chart.yaml`, `values.yaml` or `*.gotmpl`: runs `helm-docs`, regenerates the `## Charts` list in the root `README.md`, auto-commits with `[skip ci]`.
- **`remove-chart.yml`**: manual `workflow_dispatch` that removes a chart from the `gh-pages` index and its releases.
- **Renovate** extends `github>tibuntu/renovate-presets` (do not assume the stock Renovate defaults). `ignorePaths` is empty so `templates/tests` is scanned; regex managers in `renovate.json` bump the whoami image in `charts/*/ci/*-values.yaml` and busybox in the test pods.

## Chart conventions

- **Values live in three places:** `values.yaml` (every value needs a `# -- ` comment; helm-docs builds the README from it and `README.md.gotmpl`), `values.schema.json`, and the template. The schema is strict (`additionalProperties: false`): a new value missing from it fails installs. Pass-through Kubernetes objects stay open.
- **Image:** `registry/repository:tag`. `image.registry` and `image.repository` are enforced by `required` in the template, not the schema, so `helm lint` with defaults stays green.
- **Security defaults:** UID/GID 911, meets Pod Security Standard `restricted` (runAsNonRoot, readOnlyRootFilesystem, no privilege escalation, drop ALL, seccomp RuntimeDefault). Helm merges maps, so a partial `securityContext` override keeps the other keys. The service account token is not mounted unless `serviceAccount.automount: true`.
- **`environment`:** a list in Kubernetes env format (`name` + `value`/`valueFrom`), default `[]`. The schema rejects non-empty maps but accepts `{}` for old files. `envFrom` is passed to the container.
- **Ports:** `containerPort` defaults to `service.port`; the named port `http` follows it and the Service `targetPort` is `http`. No `ExternalName` service type.
- **Probes:** liveness, readiness and startup, `httpGet` on port `http` by default; any other probe field passes through.
- **Persistence:** PVC name is `persistence.existingClaim`, else `persistence.name`, else `<fullname>-pvc`; `existingClaim` skips PVC creation. `persistence.annotations` (e.g. `helm.sh/resource-policy: keep`) and per-entry annotations on `additionalPersistentVolumeClaims` are supported.
- **Chart testing:** `charts/*/ci/*-values.yaml` are the values `ct` tests with. `.helmignore` keeps `ci/` out of the package. A `helm test` pod lives in `templates/tests` (`nc -z` to the Service).
- **Ingress:** `networking.k8s.io/v1` only; `kubeVersion` requires 1.22+.

## Adding a new chart

1. Create `charts/<name>/` with `Chart.yaml` (including `kubeVersion`, `maintainers`, `sources`, and `home: https://github.com/tibuntu/helm-charts/tree/main/charts/<name>`), `values.yaml`, `values.schema.json` (`additionalProperties: false`), `README.md.gotmpl`, `.helmignore`, `templates/` with `_helpers.tpl`, and `ci/*-values.yaml` for `ct`.
2. The root `README.md` chart list updates on the next push via `docs.yml`.

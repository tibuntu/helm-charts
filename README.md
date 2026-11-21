# Helm Charts

[![License][license-img]][license]

[license-img]: https://img.shields.io/badge/License-Apache%202.0-blue.svg
[license]: https://github.com/tibuntu/helm-charts/blob/main/LICENSE

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/tibuntu)](https://artifacthub.io/packages/search?repo=tibuntu)

Helm charts for running applications on a [Kubernetes](https://kubernetes.io)
cluster, installed with the [Helm](https://helm.sh) package manager.

## Charts

- [generic](https://github.com/tibuntu/helm-charts/tree/main/charts/generic) - A Helm chart for generic applications

All my charts are also available via [Artifact Hub](https://artifacthub.io/packages/search?repo=tibuntu).

## Installing Helm

Follow the [official install guide](https://helm.sh/docs/intro/install/). Helm 3 or later is required, and the charts need Kubernetes 1.22 or later.

## Installing Tibuntu Helm Repository

```bash
helm repo add tibuntu https://tibuntu.github.io/helm-charts
helm repo update
```

## Installing a Chart

Search all the repositories available

```bash
helm search repo tibuntu -l
```

Install the `generic` chart. It needs an image: set both `image.registry` and `image.repository`, and the tag if you do not want `latest`.

```bash
helm install my-app tibuntu/generic \
  --set image.registry=docker.io \
  --set image.repository=traefik/whoami \
  --set image.tag=v1.12.0
helm status my-app
```

The charts are also published to the OCI registry `ghcr.io/tibuntu/charts`, which needs no `helm repo add`:

```bash
helm install my-app oci://ghcr.io/tibuntu/charts/generic \
  --set image.registry=docker.io \
  --set image.repository=traefik/whoami \
  --set image.tag=v1.12.0
```

## Running Voyager

[Voyager](https://github.com/aeharding/voyager), a web client for Lemmy, runs on the `generic` chart. The `voyager` chart is no longer maintained; its published versions stay installable from the repository.

Save these values as `voyager-values.yaml`:

```yaml
# Names and labels say "voyager" like the old voyager chart did: release
# "voyager" gets a Deployment and Service named "voyager" (not "voyager-generic").
nameOverride: voyager

image:
  registry: ghcr.io
  repository: aeharding/voyager
  tag: "2.49.1"

service:
  port: 5314

livenessProbe:
  enabled: true
readinessProbe:
  enabled: true

resources:
  requests:
    cpu: 15m
    memory: 50M
  limits:
    cpu: 200m
    memory: 350M

# The image is nginx. With the chart's read-only root filesystem it needs these
# writable: nginx temp files, the pid file, and the server config it renders at startup.
extraEmptyDirs:
  - name: nginx-cache
    mountPath: /var/cache/nginx
  - name: nginx-run
    mountPath: /run
  - name: nginx-conf
    mountPath: /etc/nginx/conf.d

# Custom Lemmy servers: the image's CUSTOM_LEMMY_SERVERS variable
# (https://github.com/aeharding/voyager#environment-variables) has no effect here,
# because its startup script writes /var/www/config.json, which the non-root
# user cannot. Serve that file from a pod annotation instead:
# podAnnotations:
#   voyager/config: '{ "customServers": ["lemmy.world", "lemmy.ml", "sh.itjust.works"] }'
# additionalVolumes:
#   - name: config
#     downwardAPI:
#       items:
#         - path: config.json
#           fieldRef:
#             fieldPath: metadata.annotations['voyager/config']
# additionalVolumeMounts:
#   - name: config
#     mountPath: /var/www/config.json
#     subPath: config.json
```

```bash
helm install voyager tibuntu/generic -f voyager-values.yaml
```

An existing release of the `voyager` chart moves over in place with `helm upgrade voyager tibuntu/generic -f voyager-values.yaml`: resource names and selector labels stay the same. Carry over your own values (ingress, CUSTOM_LEMMY_SERVERS as shown above).

## Uninstalling the Chart

To uninstall the `my-app` release:

```bash
helm uninstall my-app
```

## Development

Install the git hooks once, then run the checks before pushing:

```bash
pre-commit install
pre-commit run --all-files
```

Chart READMEs are generated from `values.yaml` by helm-docs:

```bash
helm-docs --chart-search-root=charts
```

Releases are automatic. To release a chart, bump `version` in its `Chart.yaml` and merge to `main`.

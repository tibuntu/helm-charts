# Helm Charts

[![License][license-img]][license]

[license-img]: https://img.shields.io/badge/License-Apache%202.0-blue.svg
[license]: https://github.com/tibuntu/helm-charts/blob/main/LICENSE

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/tibuntu)](https://artifacthub.io/packages/search?repo=tibuntu)

Tibuntu Helm Charts on a [Kubernetes](https://kubernetes.io) cluster using the
[Helm](https://helm.sh) package manager.

## Charts

- [Voyager](https://github.com/tibuntu/helm-charts/tree/main/charts/voyager)
- [unpoller](https://github.com/tibuntu/helm-charts/tree/main/charts/unpoller)

All my charts are also available via [ArtifactHub](https://artifacthub.io/packages/search?user=tibuntu)

## Installing [Helm](https://helm.sh)

```bash
curl -L https://git.io/get_helm.sh | bash
helm init
```

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

Install specific helm chart

```bash
helm install voyager tibuntu/voyager
helm status voyager
```

## Uninstalling the Chart

To uninstall/delete the `voyager` deployment:

```bash
helm delete voyager
```

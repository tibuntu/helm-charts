# Helm Charts

[![License][license-img]][license]

[license-img]: https://img.shields.io/badge/License-Apache%202.0-blue.svg
[license]: https://github.com/tibuntu/helm-charts/blob/main/LICENSE

Tibuntu Helm Charts on a [Kubernetes](https://kubernetes.io) cluster using the
[Helm](https://helm.sh) package manager.

## Charts

- [Voyager](https://github.com/tibuntu/helm-charts/tree/main/charts/voyager)

#### Installing [Helm](https://helm.sh)

```
curl -L https://git.io/get_helm.sh | bash
helm init
```

## Installing Tibuntu Helm Repository

```
helm repo add tibuntu https://tibuntu.github.io/helm-charts
helm repo update
```

## Installing the Chart

Search all the repositories available
```
helm search repo tibuntu -l
```

Install specific helm chart
```
helm install voyager tibuntu/voyager
helm status voyager
```

## Uninstalling the Chart

To uninstall/delete the `voyager` deployment:

```
$ helm delete voyager
```

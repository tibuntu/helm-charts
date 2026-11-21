# generic

![Version: 0.10.0](https://img.shields.io/badge/Version-0.10.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

A Helm chart for generic applications

**Homepage:** <https://github.com/tibuntu/helm-charts/tree/main/charts/generic>

## Install

```bash
helm repo add tibuntu https://tibuntu.github.io/helm-charts
helm install my-app tibuntu/generic \
  --set image.registry=docker.io \
  --set image.repository=traefik/whoami \
  --set image.tag=v1.12.0
```

`image.registry` and `image.repository` are required; rendering fails without them. Unknown keys in your values are rejected by the chart's schema, so a typo fails the install instead of being ignored.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| tibuntu |  | <https://github.com/tibuntu> |

## Source Code

* <https://github.com/tibuntu/helm-charts>

## Requirements

Kubernetes: `>=1.22.0-0`

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| additionalPersistentVolumeClaims | list | `[]` | Additional PersistentVolumeClaims to create and mount, named `<fullname>-<name>`. Per entry: `name`, `mountPath`, `size` (default 1Gi), `accessMode` (default ReadWriteOnce), `storageClass`, `annotations` |
| additionalVolumeMounts | list | `[]` | Additional volume mounts to add to the container (raw Kubernetes volumeMount specs) |
| additionalVolumes | list | `[]` | Additional volumes to add to the pod (raw Kubernetes volume specs) |
| affinity | object | `{}` | Affinity rules for the pod |
| args | list | `[]` | Override the image CMD. Leave empty to keep the image default. Set this (not `command`) to run another process through the image's entrypoint script, e.g. a background worker sharing the image of a web app. |
| command | list | `[]` | Override the image ENTRYPOINT. Leave empty to keep the image default. |
| containerPort | string | `""` | Port the container listens on (the `http` port); empty means same as `service.port` |
| envFrom | list | `[]` | Sources for environment variables, passed to the container as `envFrom` (configMapRef / secretRef) |
| environment | list | `[]` | Custom environment variables, a list in the Kubernetes `env` format (`name` with `value` or `valueFrom`) |
| extraEmptyDirs | list | `[]` | Mount additional emptyDir volumes (list of `name` and `mountPath`), e.g. a writable /tmp while `readOnlyRootFilesystem` is on |
| fullnameOverride | string | `""` | Override the full resource name (replaces `<release>-<chart>`) |
| image.pullPolicy | string | `"IfNotPresent"` | Image pull policy (Always, IfNotPresent or Never) |
| image.registry | string | `nil` | Image registry host, e.g. `docker.io`. Required: rendering fails when unset |
| image.repository | string | `nil` | Image repository, e.g. `traefik/whoami`. Required: rendering fails when unset |
| image.tag | string | `"latest"` | Image tag; falls back to the chart appVersion when empty |
| imagePullSecrets | list | `[]` | Secrets used to pull the image, a list of `name:` references to existing Secrets |
| ingress.annotations | object | `{}` | Annotations for the Ingress |
| ingress.className | string | `""` | `ingressClassName` of the Ingress; omitted when empty |
| ingress.enabled | bool | `false` | Create an Ingress |
| ingress.hosts | list | `[{"host":"my-subdomain.my-domain.com","paths":[{"path":"/","pathType":"ImplementationSpecific"}]}]` | Ingress rules; every path routes to this chart's Service on `service.port`. `pathType` defaults to ImplementationSpecific |
| ingress.tls | list | `[]` | Ingress TLS entries (`hosts` and an optional `secretName`) |
| livenessProbe | object | `{"enabled":false,"initialDelaySeconds":10,"path":"/","port":"http"}` | Liveness probe, httpGet; any other probe field (periodSeconds, ...) is passed through |
| livenessProbe.enabled | bool | `false` | Render the probe on the container |
| livenessProbe.initialDelaySeconds | int | `10` | Seconds to wait after the container starts before probing |
| livenessProbe.path | string | `"/"` | HTTP path to request |
| livenessProbe.port | string | `"http"` | Port number or container port name (`http` follows the container port) |
| nameOverride | string | `""` | Override the chart name used in resource names and the `app.kubernetes.io/name` label |
| nodeSelector | object | `{}` | Node selector for the pod |
| persistence.accessMode | string | `"ReadWriteOnce"` | Access mode of the PVC (ReadWriteOnce, ReadOnlyMany, ReadWriteMany or ReadWriteOncePod) |
| persistence.annotations | object | `{}` | Annotations for the PVC, e.g. `helm.sh/resource-policy: keep` to retain the PVC (and its data) on `helm uninstall` |
| persistence.enabled | bool | `false` | Mount a PersistentVolumeClaim (created unless `existingClaim` is set) |
| persistence.existingClaim | string | `""` | Name of an existing PVC to mount instead of creating one. When set, no PVC is created. |
| persistence.mountPath | string | `"/data"` | Mount path of the PVC in the container |
| persistence.name | string | `""` | Name of the PVC; defaults to `<fullname>-pvc`. Ignored when `existingClaim` is set |
| persistence.size | string | `"1Gi"` | Storage size requested by the PVC |
| persistence.storageClass | string | `""` | Storage class name; the cluster default storage class is used when empty |
| podAnnotations | object | `{}` | Annotations for the pod template |
| podSecurityContext | object | `{"fsGroup":911}` | Pod-level securityContext. `fsGroup` makes mounted volumes writable for the container user |
| priorityClassName | string | `""` | PriorityClass for the pod, e.g. so a pod pinned to one node can preempt others |
| readinessProbe | object | `{"enabled":false,"initialDelaySeconds":10,"path":"/","port":"http"}` | Readiness probe, httpGet; any other probe field (periodSeconds, ...) is passed through |
| readinessProbe.enabled | bool | `false` | Render the probe on the container |
| readinessProbe.initialDelaySeconds | int | `10` | Seconds to wait after the container starts before probing |
| readinessProbe.path | string | `"/"` | HTTP path to request |
| readinessProbe.port | string | `"http"` | Port number or container port name (`http` follows the container port) |
| replicaCount | int | `1` | Number of pod replicas of the Deployment |
| resources | object | `{"limits":{},"requests":{}}` | Container resource requests and limits (passed as `resources`) |
| securityContext | object | `{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":true,"runAsGroup":911,"runAsNonRoot":true,"runAsUser":911,"seccompProfile":{"type":"RuntimeDefault"}}` | Container securityContext; defaults follow the restricted Pod Security Standard. Overriding it replaces the whole map, so repeat the keys you want to keep |
| service.annotations | object | `{}` | Annotations for the Service (e.g. Tailscale, MetalLB, external-dns) |
| service.port | int | `5314` | Service port; also the container port unless `containerPort` is set |
| service.type | string | `"ClusterIP"` | Service type (ClusterIP, NodePort or LoadBalancer) |
| serviceAccount.annotations | object | `{}` | Annotations for the ServiceAccount (e.g. IAM role bindings); only used when `create` is true |
| serviceAccount.automount | bool | `true` | Mount the service account token into the pod (`automountServiceAccountToken`); set to false if the workload does not call the Kubernetes API |
| serviceAccount.create | bool | `true` | Create a ServiceAccount; when false the pod uses `name` or the namespace `default` account |
| serviceAccount.name | string | `""` | ServiceAccount name; defaults to the full resource name when `create` is true, otherwise to `default` |
| startupProbe | object | `{"enabled":false,"initialDelaySeconds":10,"path":"/","port":"http"}` | Startup probe, httpGet; any other probe field (periodSeconds, ...) is passed through |
| startupProbe.enabled | bool | `false` | Render the probe on the container |
| startupProbe.initialDelaySeconds | int | `10` | Seconds to wait after the container starts before probing |
| startupProbe.path | string | `"/"` | HTTP path to request |
| startupProbe.port | string | `"http"` | Port number or container port name (`http` follows the container port) |
| strategy | object | `{}` | Deployment update strategy. Leave empty to use the Kubernetes default (RollingUpdate). Set `type: Recreate` to terminate all existing pods before creating new ones — useful for workloads backed by a ReadWriteOnce volume. |
| tolerations | list | `[]` | Tolerations for the pod |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)

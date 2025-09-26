# MinIO Helm Chart

A Helm chart for deploying MinIO on Kubernetes

![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.0.0](https://img.shields.io/badge/AppVersion-1.0.0-informational?style=flat-square)

**Homepage:** <https://github.com/deathbymisadventure/minio-helm>

## About MinIO

MinIO is an S3 compatible file storage system.

### Key Features

- Exabyte Scale: Supports frictionless growth in a single flat namespace, enabling seamless expansion for large-scale data storage needs.
- Warp Speed Performance: Delivers 21.8 TiB/s throughput, saturating hardware from drive IOPS to network throughput with zero bottlenecks, ideal for high-performance requirements.
- Deepest AI Ecosystem: Offers native integration with PyTorch, Iceberg, and every major AI framework, along with the richest S3 API support.
- Zero Lock-In: Provides flexibility from edge to core to cloud, ensuring compatibility across different environments without vendor lock-in.

### Benefits

- Unmatched Performance for AI: Enables fast AI training pipelines and real-time data for autonomous agents, scaling with GPU speed for AI models and applications.
Cost Efficiency: Achieves 40% lower TCO with best-in-class unit economics, reducing operational costs for enterprises.
- Wide Adoption: Trusted by 77% of the Fortune 500, 9/10 of the largest automotive companies, 10/10 of the largest US banks, and 8/10 of the largest US retailers, indicating reliability and scalability.
- Future-Proof Infrastructure: Unifies and future-proofs enterprise AI data infrastructure with a software-defined, object-native architecture, suitable for exascale era demands.
- Versatile Use Cases: Supports AI models, AI agents, and data lakehouse analytics, streaming insights instantly with compatibility for major lakehouse engines and open table formats.

## Prerequisites

- A Kubernetes cluster (version >1.19).
- Helm 3 installed.
- `kubectl` configured to interact with your cluster.
- A default StorageClass configured in the cluster for dynamic PVC provisioning.

## Configure values.yaml or valuesoverride.yaml

Customize the deployment by editing `values.yaml` or creating a `local-values.yaml` file with overrides. Key configurations include:

- `minio.image`: Specify the MinIO image repository, tag, and pull policy.
- `minio.pvc`: Configure the PVC for `/root/.MinIO` (storage size, storage class, or selector for pre-provisioned PV).
- `ingress`: Enable and configure Ingress for external access.
- `resources` and `securityContext`: Define resource limits and security settings.

## Dynamic PVC Provisioning

This chart uses dynamic provisioning for the PVC by default, with `minio.pvc.storageClassName` set to `""` to use the cluster's default StorageClass. Ensure your cluster has a default StorageClass configured:

```bash
kubectl get storageclass
```

Look for a StorageClass with `(default)` in the output. If none exists, create or annotate a StorageClass as default:

```bash
kubectl patch storageclass <storage-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

If you prefer a specific StorageClass, set `minio.pvc.storageClassName` to its name in `values.yaml`.

## Enabling Ingress

If you lack an Ingress controller, enable `ingress.enabled` in `values.yaml` to use an external Ingress solution or configure one separately. For NGINX Ingress, set `ingress.className` to `nginx` and provide TLS settings if needed.

To use a TLS certificate, create a secret:

```bash
kubectl create --namespace MinIO secret tls MinIO-tls-cert --cert=tls.crt --key=tls.key
```

Then configure `ingress.tls` in `values.yaml`:

```yaml
ingress:
  enabled: true
  hostname: minio.lvh.me
  tls:
    - secretName: MinIO-tls-cert
      hosts:
        - minio.lvh.me
```

## Deployment

1. Verify your cluster's default StorageClass:

```bash
kubectl get storageclass
```

Ensure a default StorageClass is set, or the PVC will fail to bind.

1. Create a namespace if needed:

```bash
kubectl create ns MinIO
```

1. Customize values by creating a `local-values.yaml` file with overrides from `values.yaml`.

1. Install the Helm chart:

```bash
helm install --namespace MinIO -f ./local-values.yaml minio .
```

1. Access MinIO via the Ingress hostname (e.g., `http://minio.lvh.me`) or use `kubectl port-forward` for testing:

```bash
kubectl port-forward svc/minio-minio 3000:3000 -n minio
```

Then open `http://localhost:3000` in your browser.

## Troubleshooting PVC Binding Issues

If you encounter the error `pod has unbound immediate PersistentVolumeClaims`, it means the PVC cannot bind to a Persistent Volume. To resolve:

1. **Verify Default StorageClass**:
   - Run `kubectl get storageclass` to confirm a default StorageClass exists (marked with `(default)`).
   - If none is set, annotate an existing StorageClass as default:

```bash
kubectl patch storageclass <storage-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

- Alternatively, set `minio.pvc.storageClassName` to a specific StorageClass name in `values.yaml`.

1. **Check PVC Status**:
   - Verify the PVC is bound:

```bash
kubectl get pvc -n MinIO
```

- If the PVC is `Pending`, describe it for errors:

```bash
kubectl describe pvc MinIO-MinIO-pvc -n MinIO
```

1. **Check Pod Events**:
   - Describe the pod to diagnose further:

```bash
kubectl describe pod -l app=MinIO -n MinIO
```

1. **Use a Pre-Provisioned PV (Optional)**:
   - If dynamic provisioning is unavailable, create a PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: minio-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /tmp/minio-pv
```

- Apply it:

```bash
kubectl apply -f pv.yaml
```

- Set `minio.pvc.storageClassName: standard` and `minio.pvc.selector: MinIO-pv` in `values.yaml`.

## Values

| Key | Type | Description |
|-----|------|-------------|
| imagePullSecrets | list | Secrets for private image registries |
| ingress | object | Ingress configuration |
| ingress.annotations | object | Ingress annotations |
| ingress.apiHost | string | API external hostname |
| ingress.className | string | Ingress class type |
| ingress.consoleHost | string | Console external hostname |
| ingress.enabled | bool | Enable Ingress |
| ingress.tls | object | TLS configuration with a secret |
| minio.adminPassword | string | MinIO admin password (must be at least 8 characters) |
| minio.adminUser | string | MinIO admin username |
| minio.image.pullPolicy | string | Image pull policy |
| minio.image.repository | string | Image repository |
| minio.image.tag | string | Image tag (defaults to Chart AppVersion) |
| minio.name | string | Container name |
| minio.probes | object | Health probe configurations for MinIO |
| minio.probes.enabled | bool | Enable or disable health probes |
| minio.probes.liveness | object | Liveness probe to detect if MinIO is running and responsive |
| minio.probes.liveness.failureThreshold | int | Number of consecutive failures before pod is restarted |
| minio.probes.liveness.initialDelaySeconds | int | Delay before starting probe checks (seconds) |
| minio.probes.liveness.periodSeconds | int | Interval between probe checks (seconds) |
| minio.probes.liveness.successThreshold | int | Number of consecutive successes to pass the probe |
| minio.probes.liveness.timeoutSeconds | int | Timeout for each probe attempt (seconds) |
| minio.probes.readiness | object | Readiness probe to determine if MinIO can accept traffic |
| minio.probes.readiness.failureThreshold | int | Number of consecutive failures before pod is considered unready |
| minio.probes.readiness.initialDelaySeconds | int | Delay before starting probe checks (seconds) |
| minio.probes.readiness.periodSeconds | int | Interval between probe checks (seconds) |
| minio.probes.readiness.successThreshold | int | Number of consecutive successes to pass the probe |
| minio.probes.readiness.timeoutSeconds | int | Timeout for each probe attempt (seconds) |
| minio.probes.startup | object | Startup probe to ensure MinIO initializes properly |
| minio.probes.startup.failureThreshold | int | Number of consecutive failures before pod is considered unready |
| minio.probes.startup.initialDelaySeconds | int | Delay before starting probe checks (seconds) |
| minio.probes.startup.periodSeconds | int | Interval between probe checks (seconds) |
| minio.probes.startup.successThreshold | int | Number of consecutive successes to pass the probe |
| minio.probes.startup.timeoutSeconds | int | Timeout for each probe attempt (seconds) |
| minio.pvc.selector | string | Selector to match pre-provisioned PV |
| minio.pvc.storageClassName | string | Storage Class Name for PVC or pre-provisioned PV |
| minio.pvc.storageRequest | string | PVC size for /root/.minio |
| minio.replicas | int | Number of pod replicas |
| minio.resources | object | Pod assigned resources |
| minio.securityContext | object | Pod security context |
| minio.service.consolePort | int | Console port number |
| minio.service.port | int | Service port number |
| minio.service.type | string | Service type (e.g., ClusterIP) |

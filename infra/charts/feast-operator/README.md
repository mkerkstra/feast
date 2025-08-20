# Feast Operator Helm Chart

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/feast-operator)](https://artifacthub.io/packages/search?repo=feast-operator)
[![Helm Version](https://img.shields.io/badge/helm-v3.2%2B-blue.svg)](https://helm.sh/)
[![Kubernetes Version](https://img.shields.io/badge/kubernetes-v1.19%2B-blue.svg)](https://kubernetes.io/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Feast Version](https://img.shields.io/badge/feast-v0.52.0-orange.svg)](https://feast.dev/)

This Helm chart deploys the [Feast Operator](https://github.com/feast-dev/feast/tree/master/infra/feast-operator) to a Kubernetes cluster.

![Feast Logo](https://feast.dev/feast-icon-black.png)

## Overview

The **Feast Operator** is a Kubernetes operator that manages [Feast](https://feast.dev/) feature stores. Feast is an open-source feature store for machine learning that allows teams to:

- 🏪 **Centralize feature definitions** - Define features once and reuse them across training and serving
- ⚡ **Serve features at scale** - Low-latency online serving and batch feature retrieval
- 🔄 **Keep features fresh** - Automatic feature ingestion and updates
- 📊 **Track feature lineage** - Monitor feature usage and data quality

The operator automates the deployment and management of Feast components including:
- Feature servers (online serving)
- Offline stores (training data)
- Registry services
- UI components
- Monitoring and metrics

## ✨ Key Features

- 🚀 **One-click deployment** of complete Feast infrastructure
- 🔧 **Declarative configuration** via Kubernetes Custom Resources
- 📈 **Built-in monitoring** and metrics collection
- 🔒 **Security-first** with configurable RBAC and security contexts
- 🏗️ **Production-ready** with high availability and scaling options
- 🔌 **Extensible** support for multiple storage backends

## Prerequisites

| Component | Version |
|-----------|---------|
| Kubernetes | `1.19+` |
| Helm | `3.2.0+` |
| kubectl | `1.19+` |

## 🚀 Quick Start

### Add the Helm Repository

```bash
helm repo add feast-operator https://feast-dev.github.io/feast-operator
helm repo update
```

### Install the Operator

```bash
helm install feast-operator feast-operator/feast-operator
```

### Create a FeatureStore

```bash
kubectl apply -f - <<EOF
apiVersion: feast.dev/v1alpha1
kind: FeatureStore
metadata:
  name: my-feast
spec:
  feastProject: my-project
EOF
```

### Verify Installation

```bash
# Check operator status
kubectl get deployment feast-operator-controller-manager

# Check your FeatureStore
kubectl get featurestores
```

## 🏭 Production Deployment Options

For production deployments, this chart provides several deployment patterns following the [Feast Production Guide](https://docs.feast.dev/how-to-guides/running-feast-in-production):

### Option 1: Operator + Manual FeatureStore CRs (Recommended)

```bash
# Install the operator
helm install feast-operator feast-operator/feast-operator

# Create your own FeatureStore CRs
kubectl apply -f my-featurestore.yaml
```

### Option 2: All-in-One with Example FeatureStore

```bash
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.feastProject="production" \
  --set featureStore.services.ui.enabled=true
```

### Option 3: Production with Persistence

```bash
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.quickStart.withPersistence=true \
  --set featureStore.services.onlineStore.enabled=true \
  --set featureStore.services.offlineStore.enabled=true
```

### Option 4: Development with TLS Disabled

```bash
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.quickStart.disableTLS=true \
  --set featureStore.services.ui.enabled=true
```

> **💡 Tip:** For production, manually create FeatureStore CRs with specific database connections, storage backends, and scaling configuration rather than using the built-in examples.

## Installing the Chart

### From Helm Repository (Recommended)

```bash
helm repo add feast-operator https://feast-dev.github.io/feast-operator
helm install feast-operator feast-operator/feast-operator
```

### From Source

```bash
git clone https://github.com/feast-dev/feast.git
cd feast/infra/charts/feast-operator
helm install feast-operator .
```

### With Custom Values

```bash
helm install feast-operator feast-operator/feast-operator \
  --set operator.image.tag=v0.52.0 \
  --set operator.replicaCount=2 \
  --values my-values.yaml
```

## Uninstalling the Chart

```bash
helm delete feast-operator
```

> **Note:** This will remove all Kubernetes resources created by the chart. FeatureStore custom resources will be cleaned up automatically.

## Configuration

The following table lists the configurable parameters of the Feast Operator chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `operator.image.repository` | Operator image repository | `feast-operator` |
| `operator.image.tag` | Operator image tag | `latest` |
| `operator.image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `operator.replicaCount` | Number of operator replicas | `1` |
| `operator.resources.limits.cpu` | CPU limit | `1000m` |
| `operator.resources.limits.memory` | Memory limit | `256Mi` |
| `operator.resources.requests.cpu` | CPU request | `10m` |
| `operator.resources.requests.memory` | Memory request | `64Mi` |
| `operator.env.relatedImageFeatureServer` | Feature server image | `feast:latest` |
| `operator.env.relatedImageCronJob` | Cron job image | `origin-cli:latest` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `rbac.create` | Create RBAC resources | `true` |
| `metrics.enabled` | Enable metrics service | `true` |
| `metrics.service.type` | Metrics service type | `ClusterIP` |
| `metrics.service.port` | Metrics service port | `8443` |
| `namespace.create` | Create namespace | `true` |
| `namespace.name` | Namespace name | `""` (uses Release.Namespace) |
| `crds.install` | Install CRDs | `true` |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
helm install feast-operator ./feast-operator \
  --set operator.image.tag=v0.52.0 \
  --set operator.replicaCount=2
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example:

```bash
helm install feast-operator ./feast-operator -f values.yaml
```

## Examples

### Basic Installation

```bash
helm install feast-operator ./feast-operator
```

### Custom Image

```bash
helm install feast-operator ./feast-operator \
  --set operator.image.repository=my-registry/feast-operator \
  --set operator.image.tag=v0.52.0
```

### Disable CRD Installation

If you want to install CRDs separately:

```bash
helm install feast-operator ./feast-operator --set crds.install=false
```

### Custom Resource Limits

```bash
helm install feast-operator ./feast-operator \
  --set operator.resources.limits.cpu=2000m \
  --set operator.resources.limits.memory=512Mi
```

## Creating a FeatureStore

After installing the operator, you can create a FeatureStore custom resource:

```yaml
apiVersion: feast.dev/v1alpha1
kind: FeatureStore
metadata:
  name: sample-feast
spec:
  feastProject: my-project
```

Apply this configuration:

```bash
kubectl apply -f featurestore.yaml
```

## Troubleshooting

### Check Operator Status

```bash
kubectl get deployment feast-operator-controller-manager
```

### View Operator Logs

```bash
kubectl logs deployment/feast-operator-controller-manager
```

### Check FeatureStore Status

```bash
kubectl get featurestores
kubectl describe featurestore sample-feast
```

## 📚 Additional Resources

- **[Feast Documentation](https://docs.feast.dev/)** - Complete documentation for Feast
- **[Operator Documentation](https://github.com/feast-dev/feast/tree/master/infra/feast-operator)** - Feast Operator specific docs
- **[Community Slack](https://slack.feast.dev/)** - Join the Feast community discussions
- **[Blog & Tutorials](https://feast.dev/blog/)** - Learn from real-world examples

## 🔄 Version Compatibility

| Helm Chart Version | Feast Version | Kubernetes Version |
|-------------------|---------------|-------------------|
| 0.52.x | 0.52.x | 1.19+ |

## 🏷️ Versioning

This chart follows [Semantic Versioning](https://semver.org/). Version numbers are synchronized with Feast releases.

## 📄 License

This Helm chart is licensed under the [Apache 2.0 License](https://github.com/feast-dev/feast/blob/master/LICENSE).

## Contributing

We welcome contributions! Please see the [Feast contributing guide](https://docs.feast.dev/contributing/development-guide) for details on how to contribute to this project.

### Reporting Issues

Found a bug? Please file an issue in the [Feast repository](https://github.com/feast-dev/feast/issues) with:
- Chart version
- Kubernetes version
- Helm version
- Steps to reproduce

## Support

For support, please:
1. 📖 Check the [documentation](https://docs.feast.dev/)
2. 💬 Join our [Slack community](https://slack.feast.dev/)
3. 🐛 File an issue on [GitHub](https://github.com/feast-dev/feast/issues)

---

<p align="center">
  Made with ❤️ by the <a href="https://feast.dev/community/">Feast Community</a>
</p>
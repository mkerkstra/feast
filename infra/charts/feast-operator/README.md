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

## 📋 Configuration

### Operator Configuration

The following table lists the main configurable parameters of the Feast Operator:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `operator.image.repository` | Operator image repository | `quay.io/feastdev/feast-operator` |
| `operator.image.tag` | Operator image tag | `0.52.0` |
| `operator.replicaCount` | Number of operator replicas | `1` |
| `operator.resources.limits.cpu` | CPU limit | `1000m` |
| `operator.resources.limits.memory` | Memory limit | `256Mi` |
| `namePrefix` | Resource name prefix (matches kustomize) | `feast-operator-` |
| `metrics.enabled` | Enable metrics service | `true` |
| `prometheus.serviceMonitor.enabled` | Create ServiceMonitor for Prometheus | `false` |
| `storage.defaultStorageClass` | Default storage class for all PVCs | `""` |
| `storage.defaultAccessModes` | Default access modes for PVCs | `["ReadWriteOnce"]` |

### FeatureStore Configuration

This chart can optionally deploy example FeatureStore custom resources. The configuration supports all major Feast deployment patterns:

#### 🗂️ Registry Types
- **Local**: File-based (SQLite) or cloud storage (GCS, S3)
- **SQL**: PostgreSQL, MySQL, or other SQL databases
- **Remote**: Reference to another FeatureStore or HTTP endpoint

#### 🏪 Online Store Backends
- **File**: SQLite for development
- **Redis**: High-performance in-memory store
- **DynamoDB**: AWS managed NoSQL
- **Bigtable**: Google Cloud's NoSQL service
- **PostgreSQL**: SQL-based online store
- **Cassandra**: Distributed NoSQL database

#### 📊 Offline Store Backends
- **File**: DuckDB, Parquet files for development
- **BigQuery**: Google Cloud data warehouse
- **Snowflake**: Cloud data platform
- **Redshift**: Amazon data warehouse
- **Spark**: Distributed data processing
- **PostgreSQL**: SQL-based offline store

#### ⚡ Compute Engines
- **Local**: Basic Python transformations
- **Spark**: Distributed data processing
- **AWS Lambda**: Serverless compute
- **Ray**: Distributed Python compute
- **Snowflake**: In-database transformations

#### 💾 Storage Configuration
- **Storage Classes**: Global default or per-service storage class configuration
- **Access Modes**: Configurable PVC access modes with sensible defaults
- **Annotations**: Support for storage-specific annotations (backup policies, etc)

#### 🔐 Secret Management
- **Smart Defaults**: SecretRef keys automatically default to store type if not specified
- **Flexible Secrets**: Support for multi-key secrets (e.g., separate `postgres`, `sql` keys)
- **Environment Variables**: Integration with existing secret management systems

### Example Configurations

#### GCP Deployment with BigQuery + Bigtable
```yaml
featureStore:
  enabled: true
  services:
    offlineStore:
      enabled: true
      persistence:
        store:
          enabled: true
          type: "bigquery"
          secretRef:
            name: "gcp-config"
    onlineStore:
      enabled: true
      persistence:
        store:
          enabled: true
          type: "bigtable"
          secretRef:
            name: "gcp-config"
    registry:
      local:
        enabled: true
        persistence:
          file:
            enabled: true
            cloudPath: "gs://my-bucket/registry.pb"
```

#### AWS Deployment with DynamoDB
```yaml
featureStore:
  enabled: true
  services:
    onlineStore:
      enabled: true
      persistence:
        store:
          enabled: true
          type: "dynamodb"
          secretRef:
            name: "aws-config"
    registry:
      local:
        enabled: true
        persistence:
          file:
            enabled: true
            cloudPath: "s3://my-bucket/registry.pb"
```

#### Development with Persistence
```yaml
featureStore:
  enabled: true
  quickStart:
    withPersistence: true
    disableTLS: true
  services:
    onlineStore:
      enabled: true
    ui:
      enabled: true
```

#### Production with Custom Storage Class
```yaml
storage:
  defaultStorageClass: "fast-ssd"
  defaultAccessModes:
    - ReadWriteOnce

featureStore:
  enabled: true
  services:
    onlineStore:
      enabled: true
      persistence:
        file:
          enabled: true
          pvc:
            create:
              enabled: true
              # Inherits "fast-ssd" storage class
              resources:
                requests:
                  storage: "100Gi"
              annotations:
                backup.policy: "daily"
```

#### Multi-Database Configuration with Smart SecretRefs
```yaml
featureStore:
  enabled: true
  services:
    onlineStore:
      persistence:
        store:
          enabled: true
          type: "postgres"
          secretRef:
            name: "feast-stores"
            # key defaults to "postgres" (the store type)
    offlineStore:
      persistence:
        store:
          enabled: true
          type: "bigquery"
          secretRef:
            name: "feast-stores"
            # key defaults to "bigquery"
    registry:
      local:
        persistence:
          store:
            enabled: true
            type: "sql"
            secretRef:
              name: "feast-stores"
              # key defaults to "sql"
```

For complete configuration options, see the [values.yaml](./values.yaml) file.

## 🚀 Advanced Examples

### Enterprise Deployment with External Database

```bash
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.services.onlineStore.persistence.store.enabled=true \
  --set featureStore.services.onlineStore.persistence.store.type=postgres \
  --set featureStore.services.onlineStore.persistence.store.secretRef.name=database-config \
  --set featureStore.services.registry.local.persistence.store.enabled=true \
  --set featureStore.services.registry.local.persistence.store.type=sql \
  --set featureStore.authz.oidc.enabled=true \
  --set featureStore.authz.oidc.secretRef.name=oidc-config
```

### Multi-Cluster Registry Setup

```bash
# Primary cluster (registry server)
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.services.registry.local.server.enabled=true

# Secondary cluster (registry client)
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.services.registry.remote.enabled=true \
  --set featureStore.services.registry.remote.endpoint="https://registry.primary-cluster.com"
```

### Development with Git-based Feature Repository

```bash
helm install feast-operator feast-operator/feast-operator \
  --set featureStore.enabled=true \
  --set featureStore.feastProjectDir.git.enabled=true \
  --set featureStore.feastProjectDir.git.url="https://github.com/myorg/feature-repo.git" \
  --set featureStore.feastProjectDir.git.ref="main" \
  --set featureStore.quickStart.disableTLS=true
```

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](https://github.com/feast-dev/feast/blob/master/CONTRIBUTING.md) for details on how to submit pull requests, report issues, and contribute to the project.

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/feast-dev/feast/blob/master/LICENSE) file for details.

## 🆘 Support

- 📖 [Documentation](https://docs.feast.dev/)
- 💬 [Community Slack](https://join.slack.com/t/feastopensource/shared_invite/zt-1jw6dcmgt-8LCPn8T5XVHWYb~R~VG2BA)
- 🐛 [GitHub Issues](https://github.com/feast-dev/feast/issues)
- 📧 [Mailing List](https://groups.google.com/g/feast-discuss)

---

<p align="center">
  Made with ❤️ by the <a href="https://feast.dev/community/">Feast Community</a>
</p>
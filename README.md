# K8s FluxOps

GitOps repository for managing Kubernetes infrastructure using Flux v2.

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         GKE Cluster                                  │
│  ┌───────────────┐    ┌──────────────────────────────────────────┐  │
│  │   Flux v2     │───▶│              Applications                 │  │
│  │  (GitOps)     │    │  ┌─────────────┐  ┌─────────────────┐    │  │
│  └───────────────┘    │  │Online       │  │ Nginx           │    │  │
│         │             │  │Boutique     │  │ (via Traefik)   │    │  │
│         ▼             │  └─────────────┘  └─────────────────┘    │  │
│  ┌───────────────┐    └──────────────────────────────────────────┘  │
│  │  Weave GitOps │                                                   │
│  │  Dashboard    │    ┌──────────────────────────────────────────┐  │
│  └───────────────┘    │           Infrastructure                  │  │
│                       │  ┌─────────┐ ┌─────────┐ ┌─────────────┐ │  │
│                       │  │Traefik  │ │Rancher  │ │ Kubecost    │ │  │
│                       │  │Ingress  │ │         │ │             │ │  │
│                       │  └─────────┘ └─────────┘ └─────────────┘ │  │
│                       │  ┌─────────┐ ┌─────────┐ ┌─────────────┐ │  │
│                       │  │Grafana  │ │Velero   │ │ Hubble UI   │ │  │
│                       │  │         │ │Backups  │ │ (Cilium)    │ │  │
│                       │  └─────────┘ └─────────┘ └─────────────┘ │  │
│                       │  ┌─────────┐ ┌─────────┐ ┌─────────────┐ │  │
│                       │  │Gatekeeper│ │ Jaeger │ │   OTel      │ │  │
│                       │  │(OPA)    │ │Tracing │ │ Collector   │ │  │
│                       │  └─────────┘ └─────────┘ └─────────────┘ │  │
│                       │  ┌─────────────────────────────────────┐ │  │
│                       │  │         Kube-ops-view               │ │  │
│                       │  └─────────────────────────────────────┘ │  │
│                       └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## 📁 Repository Structure

```
k8sfluxops/
├── apps/                    # Application deployments
│   └── kustomization.yaml
├── clusters/                # Cluster-specific configurations
│   └── gke/
│       ├── kustomization.yaml
│       └── gatekeeper-policies.yaml
└── infrastructure/          # Infrastructure components
    ├── gatekeeper/          # OPA Gatekeeper + Policy Manager UI
    ├── gatekeeper-policies/ # Constraint Templates
    ├── grafana/             # Metrics visualization (GCP Monitoring)
    ├── hubble/              # Network flow visualization (Cilium)
    ├── jaeger/              # Jaeger + OpenTelemetry Collector
    ├── kube-ops-view/       # Cluster visualization
    ├── kubecost/            # Cost monitoring
    ├── nginx/               # Nginx behind Traefik
    ├── online-boutique/     # Demo microservices app
    ├── rancher/             # Cluster management
    ├── traefik/             # Ingress controller
    └── velero/              # Backup & restore
```

## 🚀 Deployed Services

| Service | URL | Credentials | Purpose |
|---------|-----|-------------|---------|
| Online Boutique | http://34.148.235.24 | - | Demo microservices app |
| Kube-ops-view | http://34.75.230.182 | - | Real-time cluster visualization |
| Kubecost | http://35.227.52.192:9003 | - | Cost monitoring & optimization |
| Rancher | https://34.73.101.111 | admin / zj10pDIkbFdWumntJqqy | Multi-cluster management |
| Weave GitOps | http://34.138.228.192:9001 | admin / admin123 | Flux dashboard |
| Velero UI | http://34.73.92.29 | admin / admin | Backup management |
| Grafana | http://35.196.7.101 | admin / admin123 | Metrics & dashboards |
| Hubble UI | http://34.73.64.220 | - | Network flow visibility |
| Traefik Dashboard | http://34.73.190.38:9000/dashboard/ | - | Ingress management |
| Nginx (via Traefik) | http://34.73.190.38 | - | Backend demo |
| Gatekeeper Policy Manager | http://34.139.100.229 | - | Admission policy visualization |
| **Jaeger UI** | http://35.237.53.70 | - | Distributed tracing |

## ⚙️ GKE Cluster Features

- **Dataplane V2** (Cilium-based networking)
- **Node Auto-Provisioning (NAP)** for auto-scaling
- **Vertical Pod Autoscaler (VPA)** enabled
- **Workload Identity** for secure GCP access
- **gVNIC** for enhanced network performance
- **Managed Prometheus** for metrics collection

## 📦 Components

### Ingress & Routing
- **Traefik** - Modern ingress controller with dashboard
  - HTTP/HTTPS entry points
  - IngressRoute CRD support
  - Kubernetes Ingress support

### Distributed Tracing
- **Jaeger** - Distributed tracing backend and UI
  - All-in-one deployment with in-memory storage
  - OTLP receiver enabled
- **OpenTelemetry Collector** - Telemetry data collection
  - Receives: OTLP, Jaeger, Zipkin protocols
  - Exports: Traces to Jaeger

### Tracing Endpoints
| Protocol | Service Endpoint |
|----------|------------------|
| OTLP gRPC | `opentelemetry-collector.observability:4317` |
| OTLP HTTP | `opentelemetry-collector.observability:4318` |
| Jaeger gRPC | `opentelemetry-collector.observability:14250` |
| Jaeger Thrift | `opentelemetry-collector.observability:14268` |
| Zipkin | `opentelemetry-collector.observability:9411` |

### Policy & Security
- **OPA Gatekeeper** - Admission controller for policy enforcement
  - Constraint Templates for reusable policies
  - Audit mode for compliance checking
  - Mutation support enabled
- **Gatekeeper Policy Manager** - Web UI for policy visualization

### Constraint Templates Installed
| Template | Purpose |
|----------|---------|
| `K8sRequiredLabels` | Require specific labels on resources |
| `K8sBlockPrivileged` | Block privileged containers |
| `K8sContainerLimits` | Require CPU/memory limits |
| `K8sBlockLatestTag` | Block `:latest` image tags |

### Observability
- **Grafana** - Metrics visualization with Google Cloud Monitoring datasource
- **Hubble UI** - Network flow visualization for Dataplane V2 (Cilium)
- **Kube-ops-view** - Real-time cluster visualization
- **Jaeger** - Distributed tracing UI

### Cost & Resource Management
- **Kubecost** - Kubernetes cost monitoring and optimization
- **VPA** - Vertical Pod Autoscaler (recommendation mode)
- **HPA** - Horizontal Pod Autoscalers for Online Boutique and Nginx

### Backup & Disaster Recovery
- **Velero** - Backup and restore with GCS backend
- **Velero UI** - Web interface for backup management

### Cluster Management
- **Rancher** - Multi-cluster Kubernetes management
- **Weave GitOps** - Flux CD dashboard

### Demo Applications
- **Online Boutique** - Google's microservices demo (10 services)
- **Nginx** - Simple web server behind Traefik (with HPA)

## 🔧 Prerequisites

- GKE cluster with Dataplane V2
- Flux v2 installed
- kubectl configured
- Git repository access

## 🚀 Getting Started

1. **Bootstrap Flux**
   ```bash
   flux bootstrap github \
     --owner=<github-username> \
     --repository=k8sfluxops \
     --path=clusters/gke \
     --personal
   ```

2. **Monitor reconciliation**
   ```bash
   flux get kustomizations -A
   ```

3. **View Flux logs**
   ```bash
   flux logs -f
   ```

## 📊 Monitoring & Dashboards

### Grafana Dashboards
Import these dashboard IDs for Kubernetes monitoring:
- `315` - Kubernetes Cluster Monitoring
- `11714` - GKE Cluster Monitoring (GCP-specific)
- `6417` - Kubernetes Cluster (Prometheus)
- `13770` - Kubernetes / Views / Pods

### Traefik Dashboard
Access at `:9000/dashboard/` to view:
- Active routers and services
- Health status
- Middleware configurations

### Hubble UI
Visualize network flows between pods:
- Service map
- Protocol breakdown
- Namespace filtering

### Jaeger UI
View distributed traces:
- Service dependency graph
- Trace timeline and spans
- Latency analysis
- Error tracking

### Gatekeeper Policy Manager
View admission policies and violations:
- Constraint Templates
- Active Constraints
- Cluster-wide violations
- Compliance status

## 🔐 Security

- **OPA Gatekeeper** for admission control policies
- **Workload Identity** enabled for Velero and Grafana
- **mTLS** for Hubble relay connections
- Services use ClusterIP where possible
- Traefik handles external traffic routing

## 📝 Adding New Services

1. Create a new directory under `infrastructure/`
2. Add required manifests:
   - `namespace.yaml`
   - `helm-release.yaml` or deployment manifests
   - `kustomization.yaml`
3. Update `infrastructure/kustomization.yaml`
4. Commit and push - Flux will reconcile automatically

## 🔄 Flux Commands

```bash
# Force reconciliation
flux reconcile kustomization infrastructure --with-source

# Check HelmReleases
flux get helmreleases -A

# Suspend a kustomization
flux suspend kustomization <name>

# Resume a kustomization
flux resume kustomization <name>
```

## 🛡️ Gatekeeper Commands

```bash
# List constraint templates
kubectl get constrainttemplates

# List constraints
kubectl get constraints

# Check violations
kubectl get constraints -o json | jq '.items[].status.violations'
```

## 📡 Instrumenting Apps for Tracing

To send traces to the OpenTelemetry Collector, set the environment variable:

```yaml
env:
  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://opentelemetry-collector.observability:4317"
  - name: OTEL_SERVICE_NAME
    value: "my-service"
```

For Jaeger native instrumentation:
```yaml
env:
  - name: JAEGER_AGENT_HOST
    value: "opentelemetry-collector.observability"
  - name: JAEGER_AGENT_PORT
    value: "6831"
```

## 📄 License

MIT

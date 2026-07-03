# aiops-platform

AIOps and MLOps platform on AKS, managed via GitOps (ArgoCD).

## What this is

Three repos, three concerns:

| Repo | Purpose |
|---|---|
| `terraform-enterprise` | Builds the infrastructure (AKS, VMs, networking) |
| `ansible-configuration` | Configures the Linux VM (tools, desktop, runner) |
| `aiops-platform` | Deploys everything that runs ON the cluster (this repo) |

## Architecture

```
AKS cluster (South India, K8s 1.33, workload identity)
├── monitoring/     Prometheus + Grafana + Alertmanager
│   ├── Metrics from every pod and node every 30s
│   ├── Threshold alerts (CPU, memory, pod health)
│   └── Predictive alerts via predict_linear() — ML-based
├── logging/        Loki + Promtail
│   ├── Logs from every container automatically
│   └── Queryable in Grafana alongside metrics
├── argocd/         GitOps controller
│   ├── Watches this repo continuously
│   ├── Auto-deploys any change you push
│   └── Auto-heals drift (selfHeal=true)
└── mlops/          MLflow + Kubeflow (Phase 2)
```

## What makes this AIOps

| Capability | Tool | How |
|---|---|---|
| Predictive alerting | Prometheus predict_linear() | Predicts disk/memory exhaustion before it happens |
| Log intelligence | Loki + LogQL | Pattern-based log querying across all pods |
| Auto-remediation | Argo Events + Workflows (coming) | Automatic response to alerts without human intervention |
| GitOps self-healing | ArgoCD selfHeal=true | Cluster always matches Git — drift auto-corrected |

## Deploy

```bash
# 1. Bootstrap namespaces
kubectl apply -f bootstrap/namespaces.yaml

# 2. Connect ArgoCD to this repo (one-time setup)
kubectl apply -f gitops/applications/

# 3. ArgoCD handles everything else automatically
```

## Stack versions

| Tool | Version | Namespace |
|---|---|---|
| Prometheus + Grafana | kube-prometheus-stack latest | monitoring |
| Loki | loki-stack latest | logging |
| ArgoCD | argo-cd latest | argocd |
| MLflow | coming | mlops |
| Kubeflow | coming | mlops |

## MLOps Phase 2

- MLflow — experiment tracking, model registry
- Kubeflow — ML pipeline orchestration (K8s-native)
- KServe — model serving as Kubernetes API
- Evidently — model drift detection

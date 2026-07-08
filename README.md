# aiops-platform

AIOps and MLOps platform on AKS, managed via GitOps (ArgoCD).

## Three repos, three concerns

| Repo | Purpose | Who touches it |
|---|---|---|
| `terraform-enterprise` | Infrastructure — AKS, VMs, networking | Platform engineer |
| `ansible-configuration` | VM configuration — tools, desktop, runner | Platform engineer |
| `aiops-platform` | Everything that runs ON the cluster | Platform/ML engineer |

## Repo structure

```
aiops-platform/
│
├── bootstrap/                   One-time cluster setup
│   └── namespaces.yaml          All namespaces defined in Git
│
├── platform/                    Tools that RUN the platform
│   ├── monitoring/
│   │   ├── helmrelease.yaml     Prometheus + Grafana + Alertmanager values
│   │   └── alert-rules.yaml    Threshold + predictive (AI) alert rules
│   └── logging/
│       └── helmrelease.yaml     Loki + Promtail values
│
├── apps/                        Workloads that run ON the platform
│   └── podinfo/                 Demo app for AIOps demonstration
│       ├── deployment.yaml
│       ├── service.yaml
│       └── servicemonitor.yaml  Prometheus scrape config
│
└── argocd/                      ArgoCD wiring (the GitOps engine)
    ├── root-app.yaml            App of Apps — bootstraps everything
    └── applications/
        ├── platform/            One Application per platform tool
        │   ├── monitoring.yaml
        │   ├── logging.yaml
        │   └── alert-rules.yaml
        └── apps/                One Application per workload
            └── podinfo.yaml
```

## Why this structure

**platform/ vs apps/** — clear separation between infrastructure tools
(Prometheus, Loki) and the workloads they monitor (podinfo, future services).
Different teams, different change rates, different owners.

**argocd/applications/platform/ vs apps/** — ArgoCD Application manifests
are separate from the actual K8s manifests. This means you can change
deployment values (in platform/) without touching ArgoCD wiring (in argocd/).

**Sync waves** — deployment order is enforced:
```
Wave 1: Prometheus + Grafana (monitoring foundation)
Wave 2: Loki + Promtail (logging)
Wave 3: Alert rules (needs Prometheus CRDs from wave 1)
Wave 4: Applications (monitored from day one)
```

## Bootstrap (after every AKS rebuild)

Handled automatically by the aks.yml pipeline bootstrap-aiops job:

```bash
# What the pipeline does automatically:
helm install argocd argo/argo-cd --namespace argocd
kubectl apply -f argocd/root-app.yaml
# ArgoCD handles everything else
```

## Adding a new tool (the GitOps way)

```
1. Add K8s manifests or Helm values to platform/<tool>/ or apps/<tool>/
2. Add an ArgoCD Application to argocd/applications/platform/ or apps/
3. git add . && git commit && git push
4. ArgoCD syncs automatically — no kubectl needed
```

## AIOps stack

| Tool | Purpose | Namespace |
|---|---|---|
| Prometheus | Metrics collection + alerting | monitoring |
| Grafana | Dashboards + visualization | monitoring |
| Alertmanager | Alert routing (email/Slack/PagerDuty) | monitoring |
| Loki | Log aggregation (ELK equivalent) | logging |
| Promtail | Log collection DaemonSet | logging |
| ArgoCD | GitOps delivery + self-healing | argocd |
| Argo Events | Alert-triggered automation (coming) | argo-events |
| Argo Workflows | Auto-remediation workflows (coming) | argo-events |

## What makes this AIOps (not just DevOps)

| Capability | Implementation |
|---|---|
| Predictive alerting | `predict_linear()` in alert-rules.yaml — fires before disk/memory exhausts |
| Log intelligence | Loki + LogQL — pattern query across all pods |
| Auto-remediation | Argo Events + Workflows (coming) — automatic response without human |
| GitOps self-healing | ArgoCD `selfHeal: true` — cluster always matches Git |
| Alert correlation | Alertmanager grouping rules (coming) |

## MLOps (Phase 2 — coming)

- MLflow — experiment tracking, model registry
- Kubeflow — ML pipeline orchestration (K8s-native)
- KServe — model serving as Kubernetes API
- Evidently — model drift detection

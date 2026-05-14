# 🚀 Install Prometheus & Grafana on GKE with Detailed Command Explanation
## Kubernetes Monitoring + Helm + Grafana + Prometheus

This guide explains:
- GKE cluster creation
- Namespace creation
- Helm setup
- Prometheus installation
- Grafana setup
- Monitoring architecture
- Every command explained word by word

---

# ☁️ Step 1 — Create GKE Cluster

## Command

```bash
gcloud container clusters create monitoring-cluster \
  --zone us-central1-a \
  --machine-type e2-standard-2 \
  --num-nodes 2
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| gcloud | Google Cloud CLI tool |
| container | GKE/Kubernetes services |
| clusters | Cluster operations |
| create | Create new cluster |
| monitoring-cluster | Cluster name |
| --zone | GCP zone |
| us-central1-a | Specific datacenter region |
| --machine-type | VM machine type |
| e2-standard-2 | 2 CPU + 8GB RAM VM |
| --num-nodes | Worker node count |
| 2 | Create 2 nodes |

---

# 🔗 Step 2 — Connect to GKE Cluster

## Command

```bash
gcloud container clusters get-credentials monitoring-cluster \
  --zone us-central1-a
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| gcloud | Google Cloud CLI |
| container | Kubernetes services |
| clusters | Cluster operations |
| get-credentials | Download cluster access config |
| monitoring-cluster | Cluster name |
| --zone | Cluster zone |
| us-central1-a | Region location |

---

# ✅ Verify Nodes

## Command

```bash
kubectl get nodes
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| get | Retrieve resources |
| nodes | Worker machines |

---

# 🌐 Step 3 — Create Namespace

## Command

```bash
kubectl create ns monitor
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| create | Create resource |
| ns | Short form of namespace |
| monitor | Namespace name |

---

# 📚 Why Namespace?

Namespace separates:
- monitoring resources
- applications
- environments

---

# 🔧 Step 4 — Install Helm

## Verify Helm

```bash
helm version
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| helm | Kubernetes package manager |
| version | Show Helm version |

---

# Install Helm

## Command

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| curl | Download content |
| URL | Helm install script |
| \| | Pipe output |
| bash | Execute script |

---

# 📚 What is Helm?

:Helm simplifies:
- Kubernetes deployments
- upgrades
- rollback
- package management

---

# 🚀 Step 5 — Add Prometheus Helm Repository

## Command

```bash
helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| helm | Helm CLI |
| repo | Repository management |
| add | Add repository |
| prometheus-community | Repository nickname |
| URL | Helm chart repository URL |

---

# 📚 What is Helm Repo?

Repository stores:
- Kubernetes charts
- reusable packages
- templates

---

# Update Helm Repo

## Command

```bash
helm repo update
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| helm | Helm CLI |
| repo | Repository operations |
| update | Download latest chart info |

---

# Verify Helm Repo

## Command

```bash
helm repo list
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| helm | Helm CLI |
| repo | Repository operations |
| list | Show repositories |

---

# 🚀 Step 6 — Install Prometheus & Grafana

## Command

```bash
helm install kube-prometheus-stack \
prometheus-community/kube-prometheus-stack \
--namespace monitor
```

---

# 🧠 Full Command Explanation

| Word | Meaning |
|---|---|
| helm | Helm package manager |
| install | Install chart |
| kube-prometheus-stack | Release name |
| prometheus-community/kube-prometheus-stack | Chart name |
| --namespace | Deploy into namespace |
| monitor | Namespace name |

---

# 📚 What Gets Installed?

| Component | Purpose |
|---|---|
| Graphan| Metrics collection |
| Prometheous| Dashboards |
| Alertmanager | Alert notifications |
| Node Exporter | Node metrics |
| kube-state-metrics | Kubernetes metrics |

---

# ⏳ Wait for Deployment

Wait:
```text
2–4 minutes
```

---

# 🚀 Step 7 — Verify Pods

## Command

```bash
kubectl --namespace monitor get pods \
-l "release=kube-prometheus-stack"
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| --namespace | Target namespace |
| monitor | Namespace name |
| get | Retrieve resources |
| pods | Running containers |
| -l | Label selector |
| release=kube-prometheus-stack | Filter by release label |

---

# Get All Pods

## Command

```bash
kubectl get po --namespace monitor
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| get | Retrieve resources |
| po | Short form of pods |
| --namespace | Namespace option |
| monitor | Namespace name |

---

# 🚀 Step 8 — Verify Services

## Command

```bash
kubectl get svc --namespace monitor
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| get | Retrieve resources |
| svc | Short form of services |
| --namespace | Namespace option |
| monitor | Namespace name |

---

# 📚 What is Service?

Service exposes:
- applications
- pods
- networking endpoints

---

# 🚀 Step 9 — Verify Deployments

## Command

```bash
kubectl get deploy --namespace monitor
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| get | Retrieve resources |
| deploy | Short form deployment |
| --namespace | Namespace option |
| monitor | Namespace name |

---

# 🚀 Step 10 — Get Grafana Admin Password

## Command

```bash
kubectl --namespace monitor get secrets \
kube-prometheus-stack-grafana \
-o jsonpath="{.data.admin-password}" \
| base64 -d ; echo
```

---

# 🧠 Full Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| --namespace | Namespace option |
| monitor | Namespace name |
| get | Retrieve resource |
| secrets | Kubernetes secrets |
| kube-prometheus-stack-grafana | Secret name |
| -o | Output format |
| jsonpath | Extract specific field |
| {.data.admin-password} | Get encoded password |
| \| | Pipe output |
| base64 | Decode encoded data |
| -d | Decode mode |
| ; echo | New line output |

---

# 🔐 Default Login

| Field | Value |
|---|---|
| Username | admin |
| Password | Retrieved secret |

---

# 🚀 Step 11 — Expose Grafana Service

## Command

```bash
kubectl expose service kube-prometheus-stack-grafana \
--type=LoadBalancer \
--target-port=3000 \
--name=grafana-ext \
--namespace monitor
```

---

# 🧠 Full Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| expose | Create service |
| service | Existing service resource |
| kube-prometheus-stack-grafana | Grafana service |
| --type | Service type |
| LoadBalancer | Create external IP |
| --target-port | Internal container port |
| 3000 | Grafana application port |
| --name | New service name |
| grafana-ext | External service name |
| --namespace | Namespace option |
| monitor | Namespace name |

---

# 🌐 Verify Service

## Command

```bash
kubectl get svc --namespace monitor
```

---

# 🌍 Access Grafana

```text
http://EXTERNAL-IP:3000
```

---

# 🚀 Step 12 — Expose Prometheus

## Command

```bash
kubectl expose service \
kube-prometheus-stack-prometheus \
--type=LoadBalancer \
--target-port=9090 \
--name=prometheus-ext \
--namespace monitor
```

---

# 🧠 Full Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| expose | Create service |
| service | Service resource |
| kube-prometheus-stack-prometheus | Prometheus service |
| --type | Service type |
| LoadBalancer | Public external access |
| --target-port | Internal port |
| 9090 | Prometheus port |
| --name | Service name |
| prometheus-ext | External service name |
| --namespace | Namespace |
| monitor | Namespace name |

---

# 🌍 Access Prometheus

```text
http://EXTERNAL-IP:9090
```

---

# 🚀 Step 13 — Verify Metrics

## Command

```bash
kubectl top nodes
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| top | Resource usage |
| nodes | Worker machines |

---

# Pod Metrics

## Command

```bash
kubectl top pods -A
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| top | Resource metrics |
| pods | Pod metrics |
| -A | All namespaces |

---

# 🚀 Step 14 — Helm Important Commands

# List Helm Releases

```bash
helm list -n monitor
```

---

# 🧠 Explanation

| Word | Meaning |
|---|---|
| helm | Helm CLI |
| list | Show releases |
| -n | Namespace |
| monitor | Namespace name |

---

# Upgrade Helm Chart

```bash
helm upgrade kube-prometheus-stack \
prometheus-community/kube-prometheus-stack \
-n monitor
```

---

# 🧠 Explanation

| Word | Meaning |
|---|---|
| helm | Helm CLI |
| upgrade | Upgrade existing release |
| kube-prometheus-stack | Release name |
| prometheus-community/kube-prometheus-stack | Chart |
| -n | Namespace |
| monitor | Namespace name |

---

# Delete Helm Release

```bash
helm uninstall kube-prometheus-stack \
-n monitor
```

---

# 🧠 Explanation

| Word | Meaning |
|---|---|
| helm | Helm CLI |
| uninstall | Remove release |
| kube-prometheus-stack | Release name |
| -n | Namespace |
| monitor | Namespace name |

---

# 🚀 Step 15 — Cleanup Resources

# Delete Namespace

```bash
kubectl delete ns monitor
```

---

# 🧠 Command Breakdown

| Word | Meaning |
|---|---|
| kubectl | Kubernetes CLI |
| delete | Remove resource |
| ns | Namespace |
| monitor | Namespace name |

---

# 🧠 Complete Monitoring Architecture

```text
Kubernetes Cluster
        ↓
Node Exporters
        ↓
Prometheus
        ↓
Metrics Storage
        ↓
Grafana
        ↓
Dashboards & Alerts
```

---

# 🚀 Real Production Monitoring Stack

| Tool | Purpose |
|---|---|
| Prometheus | Metrics |
| Grafana | Dashboards |
| Alertmanager | Alerts |
| Loki | Logs |
| Tempo | Tracing |

---

# 🔥 Most Asked Interview Questions

# Beginner

## What is Prometheus?
Metrics collection system.

---

## What is Grafana?
Visualization dashboard tool.

---

## What is Helm?
Kubernetes package manager.

---

# Intermediate

## Why use namespaces?
Isolation and organization.

---

## What is kube-prometheus-stack?
Helm chart bundle for monitoring stack.

---

# Advanced

## How does Prometheus collect metrics?
Using exporters and scraping endpoints.

---

## Difference between Prometheus and Grafana?

| Prometheus | Grafana |
|---|---|
| Collects metrics | Visualizes metrics |

---

# 📚 Best Resources

| Resource | Link |
|---|---|
| Prometheus Docs | https://prometheus.io/docs |
| Grafana Docs | https://grafana.com/docs |
| Helm Docs | https://helm.sh/docs |
| GKE Docs | https://cloud.google.com/kubernetes-engine |

---


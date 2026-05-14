# 🚀 Install Prometheus & Grafana on GKE using Helm
## Complete Kubernetes Monitoring Setup Guide

This guide explains how to:
- Create a GKE cluster
- Install Helm
- Deploy Prometheus
- Deploy Grafana
- Expose Grafana UI
- Access monitoring dashboards

---

# 📚 What are Prometheus & Grafana?

| Tool | Purpose |
|---|---|
| :Prometheus | Metrics collection |
| :Grafana | Dashboards & visualization |

---

# 🧠 Monitoring Architecture

```text
Kubernetes Cluster
        ↓
Prometheus
        ↓
Collect Metrics
        ↓
Grafana
        ↓
Visual Dashboards
```

---

# 🚀 Why Use Prometheus & Grafana?

| Benefit | Description |
|---|---|
| Cluster Monitoring | Monitor Kubernetes resources |
| Pod Metrics | CPU/memory usage |
| Alerting | Failure notifications |
| Dashboards | Beautiful visualization |
| Troubleshooting | Debug production issues |

---

# ☁️ Step 1 — Create GKE Cluster

## Create Cluster

```bash
gcloud container clusters create monitoring-cluster \
  --zone us-central1-a \
  --machine-type e2-standard-2 \
  --num-nodes 2
```

---

# 🔗 Step 2 — Connect to GKE Cluster

```bash
gcloud container clusters get-credentials monitoring-cluster \
  --zone us-central1-a
```

---

# ✅ Verify Cluster

```bash
kubectl get nodes
```

---

# 🚀 Step 3 — Create Namespace

Creating separate namespace is a good practice.

---

# Create Namespace

```bash
kubectl create ns monitor
```

---

# Verify Namespace

```bash
kubectl get ns
```

---

# 🧠 Why Namespace?

| Benefit | Description |
|---|---|
| Isolation | Separate monitoring resources |
| Security | Better RBAC |
| Organization | Cleaner cluster management |

---

# 🚀 Step 4 — Install Helm

## Verify Helm

```bash
helm version
```

---

# Install Helm (Linux)

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

# 🧠 What is Helm?

:Helm is a package manager for Kubernetes.

Helm simplifies:
- deployments
- upgrades
- rollback
- configuration

---

# 🚀 Step 5 — Add Prometheus Helm Repository

## Add Helm Repo

```bash
helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts
```

---

# Update Repo

```bash
helm repo update
```

---

# Verify Repo

```bash
helm repo list
```

---

# 🚀 Step 6 — Install Prometheus & Grafana

## Install kube-prometheus-stack

```bash
helm install kube-prometheus-stack \
prometheus-community/kube-prometheus-stack \
--namespace monitor
```

---

# 🧠 Command Explanation

| Part | Meaning |
|---|---|
| kube-prometheus-stack | Release name |
| prometheus-community/kube-prometheus-stack | Helm chart |
| --namespace monitor | Deploy into monitor namespace |

---

# 🚀 What Gets Installed?

| Component | Purpose |
|---|---|
| Prometheus | Metrics collection |
| Grafana | Dashboards |
| Alertmanager | Alerts |
| Node Exporter | Node metrics |
| kube-state-metrics | Kubernetes metrics |

---

# ⏳ Wait for Deployment

Deployment may take:
```text
2–4 minutes
```

---

# 🚀 Step 7 — Verify Pods

## Get Monitoring Pods

```bash
kubectl --namespace monitor get pods \
-l "release=kube-prometheus-stack"
```

---

# Get All Pods

```bash
kubectl get po --namespace monitor
```

---

# ✅ Expected Pods

```text
grafana
prometheus
alertmanager
node-exporter
kube-state-metrics
```

---

# 🚀 Step 8 — Verify Services

## Get Services

```bash
kubectl get svc --namespace monitor
```

---

# Expected Output

```text
grafana service
prometheus service
alertmanager service
```

---

# 🚀 Step 9 — Verify Deployments

```bash
kubectl get deploy --namespace monitor
```

---

# 🚀 Step 10 — Get Grafana Admin Password

## Retrieve Password

```bash
kubectl --namespace monitor get secrets \
kube-prometheus-stack-grafana \
-o jsonpath="{.data.admin-password}" \
| base64 -d ; echo
```

---

# 🧠 Default Grafana Username

```text
admin
```

---

# 🚀 Step 11 — Expose Grafana Service

# Check Grafana Service

```bash
kubectl get svc --namespace monitor
```

---

# Find Grafana Service

Example:

```text
kube-prometheus-stack-grafana
```

---

# 🚀 Expose Grafana as LoadBalancer

```bash
kubectl expose service kube-prometheus-stack-grafana \
--type=LoadBalancer \
--target-port=3000 \
--name=grafana-ext \
--namespace monitor
```

---

# 🧠 Why Port 3000?

Grafana runs internally on:
```text
Port 3000
```

---

# 🚀 Step 12 — Verify External Service

```bash
kubectl get svc --namespace monitor
```

---

# Expected Output

```text
NAME          TYPE           EXTERNAL-IP
grafana-ext   LoadBalancer   34.xx.xx.xx
```

---

# 🌐 Access Grafana UI

Open browser:

```text
http://EXTERNAL-IP:3000
```

---

# 🔐 Login Credentials

| Field | Value |
|---|---|
| Username | admin |
| Password | Retrieved secret |

---

# 🚀 Step 13 — Access Prometheus UI

## Expose Prometheus

```bash
kubectl expose service \
kube-prometheus-stack-prometheus \
--type=LoadBalancer \
--target-port=9090 \
--name=prometheus-ext \
--namespace monitor
```

---

# Verify Prometheus Service

```bash
kubectl get svc --namespace monitor
```

---

# 🌐 Access Prometheus

```text
http://EXTERNAL-IP:9090
```

---

# 🚀 Step 14 — Verify Metrics Collection

# Prometheus Targets

Inside Prometheus UI:

```text
Status → Targets
```

---

# Verify:

```text
UP
```

for:
- nodes
- kube-state-metrics
- prometheus
- exporters

---

# 🚀 Step 15 — Import Grafana Dashboards

## Popular Dashboards

| Dashboard | Purpose |
|---|---|
| Kubernetes Cluster | Cluster metrics |
| Node Exporter | Node monitoring |
| Pod Metrics | Pod CPU/memory |

---

# Import Dashboard

Grafana:
```text
Dashboards → Import
```

---

# Popular Dashboard IDs

| Dashboard | ID |
|---|---|
| Kubernetes Cluster Monitoring | 315 |
| Node Exporter Full | 1860 |

---

# 🚀 Step 16 — Check Resource Metrics

# Node Metrics

```bash
kubectl top nodes
```

---

# Pod Metrics

```bash
kubectl top pods -A
```

---

# 🚀 Step 17 — Monitoring Architecture

```text
Kubernetes Cluster
        ↓
Node Exporter
        ↓
Prometheus
        ↓
Grafana
        ↓
Dashboards & Alerts
```

---

# 🚀 Step 18 — Important Helm Commands

# List Releases

```bash
helm list -n monitor
```

---

# Upgrade Helm Chart

```bash
helm upgrade kube-prometheus-stack \
prometheus-community/kube-prometheus-stack \
-n monitor
```

---

# Uninstall Helm Release

```bash
helm uninstall kube-prometheus-stack \
-n monitor
```

---

# 🚀 Step 19 — Cleanup Resources

## Delete Helm Release

```bash
helm uninstall kube-prometheus-stack \
--namespace monitor
```

---

# Delete Namespace

```bash
kubectl delete ns monitor
```

---

# 🚀 Production Best Practices

- use dedicated namespace
- enable persistent storage
- configure alerts
- secure Grafana login
- enable ingress + HTTPS
- use RBAC

---

# 🚀 Recommended Production Setup

| Tool | Purpose |
|---|---|
| Prometheus | Metrics |
| Grafana | Visualization |
| Alertmanager | Alerts |
| Loki | Logs |
| Tempo | Tracing |

---

# 🚀 Real Production Monitoring Architecture

```text
Applications
      ↓
Exporters
      ↓
Prometheus
      ↓
Grafana
      ↓
Dashboards & Alerts
```

---

# 🚨 Common Problems

| Problem | Cause | Solution |
|---|---|---|
| Pods Pending | Insufficient resources | Add nodes |
| Grafana not accessible | Service not exposed | Use LoadBalancer |
| No metrics | Exporters down | Check targets |
| Wrong password | Secret issue | Regenerate secret |

---

# 🔥 Troubleshooting Commands

# Check Pods

```bash
kubectl get pods -n monitor
```

---

# Describe Pod

```bash
kubectl describe pod <pod-name> -n monitor
```

---

# View Logs

```bash
kubectl logs <pod-name> -n monitor
```

---

# Check Services

```bash
kubectl get svc -n monitor
```

---

# Check Helm Releases

```bash
helm list -n monitor
```

---

# 🔥 Most Asked Interview Questions

# Beginner

## What is Prometheus?
Metrics monitoring system.

---

## What is Grafana?
Visualization dashboard platform.

---

## What is Helm?
Kubernetes package manager.

---

# Intermediate

## What does kube-prometheus-stack install?

- Prometheus
- Grafana
- Alertmanager
- Exporters

---

## Why use namespaces?
Isolation and organization.

---

# Advanced

## How does Prometheus collect metrics?
Using exporters and scraping targets.

---

## Difference between Prometheus and Grafana?

| Prometheus | Grafana |
|---|---|
| Stores metrics | Visualizes metrics |

---

## What is Alertmanager?
Handles monitoring alerts.

---

# 📚 Best Resources

| Resource | Link |
|---|---|
| Prometheus Docs | https://prometheus.io/docs |
| Grafana Docs | https://grafana.com/docs |
| Helm Charts | https://github.com/prometheus-community/helm-charts |
| GKE Docs | https://cloud.google.com/kubernetes-engine |

---

# ⭐ Final Goal

Build:
- Enterprise Kubernetes Monitoring
- Production Observability Platforms
- Cloud Monitoring Systems
- Auto-Healing Infrastructure
- High Availability Kubernetes Clusters

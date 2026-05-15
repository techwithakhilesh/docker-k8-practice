# 🚀 Set Up Horizontal Pod Autoscaler (HPA) in GKE
## Kubernetes Auto Scaling Mastery Guide

This guide explains how to:
- Deploy Nginx on GKE
- Configure CPU resource limits
- Enable Metrics Server
- Configure Horizontal Pod Autoscaler (HPA)
- Simulate traffic load
- Observe automatic pod scaling

---

# 📚 What is HPA?

: Horizontal Pod Autoscaler (HPA) automatically scales pods based on:

- CPU usage
- Memory usage
- Custom metrics

---

# 🧠 HPA Architecture

```text
Users Traffic
      ↓
Kubernetes Service
      ↓
Deployment
      ↓
Pods
      ↓
HPA Monitors CPU
      ↓
Scale Up / Down
```

---

# 🚀 Why Use HPA?

| Benefit | Description |
|---|---|
| Auto Scaling | Automatically adjusts pods |
| Cost Optimization | Reduces unused resources |
| High Availability | Handles traffic spikes |
| Performance | Prevents overload |
| Reliability | Improves uptime |

---

# ☁️ Step 1 — Create GKE Cluster

## Create Cluster

```bash
gcloud container clusters create hpa-cluster \
  --zone us-central1-a \
  --machine-type e2-standard-2 \
  --num-nodes 2
```

---

# 🔗 Step 2 — Connect to GKE Cluster

```bash
gcloud container clusters get-credentials hpa-cluster \
  --zone us-central1-a
```

---

# ✅ Step 3 — Verify Cluster

## Check Nodes

```bash
kubectl get nodes
```

---

## Detailed Nodes

```bash
kubectl get nodes -o wide
```

---

# 🚀 Step 4 — Create Nginx Deployment YAML

## Create File

```text
nginx-deployment.yaml
```

---

# 📄 nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx

          resources:
            requests:
              cpu: "250m"

            limits:
              cpu: "500m"

          ports:
            - containerPort: 80
```

---

# 🧠 YAML Explanation

| Field | Purpose |
|---|---|
| replicas | Initial pod count |
| requests | Minimum CPU required |
| limits | Maximum CPU allowed |
| selector | Pod matching labels |
| containerPort | Exposed app port |

---

# 🚀 Step 5 — Apply Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

---

# ✅ Verify Deployment

## Get Pods

```bash
kubectl get pods
```

---

## Describe Deployment

```bash
kubectl describe deployment nginx-deployment
```

---

# 🚀 Step 6 — Expose Deployment as Service

## Create LoadBalancer Service

```bash
kubectl expose deployment nginx-deployment \
  --type=LoadBalancer \
  --port=80 \
  --name=nginx-service
```

---

# ✅ Verify Service

```bash
kubectl get svc
```

---

# 🌐 Expected Output

```text
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP
nginx-service   LoadBalancer   34.xx.xx.xx    35.xx.xx.xx
```

---

# 🚀 Step 7 — Enable Metrics Server

HPA requires Metrics Server to collect CPU metrics.

---

# Install Metrics Server

```bash
kubectl apply -f \
https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

# ✅ Verify Metrics Server

```bash
kubectl get apiservices | grep metrics
```

---

# Check Metrics Pod

```bash
kubectl get pods -n kube-system
```

---

# 🚀 Step 8 — Verify CPU Metrics

## Check Node Metrics

```bash
kubectl top nodes
```

---

## Check Pod Metrics

```bash
kubectl top pods
```

---

# 🚀 Step 9 — Create Horizontal Pod Autoscaler

## Create HPA

```bash
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=2 \
  --max=5
```

---

# 🧠 HPA Configuration Meaning

| Option | Meaning |
|---|---|
| --cpu-percent=50 | Scale when CPU > 50% |
| --min=2 | Minimum pods |
| --max=5 | Maximum pods |

---

# ✅ Verify HPA

```bash
kubectl get hpa
```

---

# Expected Output

```text
NAME               REFERENCE                     TARGETS
nginx-deployment   Deployment/nginx-deployment   0%/50%
```

---

# 🚀 Step 10 — Simulate CPU Load

# Create Load Generator Pod

```bash
kubectl run -it --rm \
  --image=busybox \
  load-generator -- /bin/sh
```

---

# Generate Continuous Traffic

Inside container:

```bash
while true; do
  wget -q -O- http://nginx-service;
done
```

---

# 🧠 What Happens?

Traffic increases:
- CPU usage
- pod load

HPA detects:
```text
CPU > 50%
```

Then automatically:
```text
Scale pods from 2 → 5
```

---

# 🚀 Step 11 — Watch Auto Scaling

## Monitor HPA

```bash
kubectl get hpa -w
```

---

## Watch Pods Live

```bash
kubectl get pods -w
```

---

# Expected Scaling

```text
2 Pods
   ↓
3 Pods
   ↓
5 Pods
```

---

# 🚀 Step 12 — Observe Resource Usage

## Pod CPU Usage

```bash
kubectl top pods
```

---

## Node Usage

```bash
kubectl top nodes
```

---

# 🚀 Step 13 — Scale Down Automatically

When traffic reduces:
- CPU usage drops
- HPA scales down pods

---

# Auto Scaling Flow

```text
High CPU
   ↓
Scale Up

Low CPU
   ↓
Scale Down
```

---

# 🚀 Step 14 — Cleanup Resources

## Delete HPA

```bash
kubectl delete hpa nginx-deployment
```

---

## Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

---

## Delete Service

```bash
kubectl delete svc nginx-service
```

---

# 🔥 Most Important HPA Commands

# Create HPA

```bash
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=2 \
  --max=5
```

---

# Get HPA

```bash
kubectl get hpa
```

---

# Describe HPA

```bash
kubectl describe hpa nginx-deployment
```

---

# Delete HPA

```bash
kubectl delete hpa nginx-deployment
```

---

# 🔥 Important Kubernetes Commands

## Get Pods

```bash
kubectl get pods
```

---

## Watch Pods

```bash
kubectl get pods -w
```

---

## Get Services

```bash
kubectl get svc
```

---

## Describe Deployment

```bash
kubectl describe deployment nginx-deployment
```

---

## Check Metrics

```bash
kubectl top pods
```

---

# 🧠 Real Production HPA Architecture

```text
Internet Traffic
       ↓
LoadBalancer
       ↓
Kubernetes Service
       ↓
Deployment
       ↓
Pods
       ↓
HPA Controller
       ↓
Auto Scaling
```

---

# 🚀 Advanced HPA Features

| Feature | Purpose |
|---|---|
| CPU Scaling | Scale on CPU |
| Memory Scaling | Scale on RAM |
| Custom Metrics | Scale using APIs |
| External Metrics | Cloud monitoring |

---

# 🚀 HPA Best Practices

- always define resource requests
- use Metrics Server
- avoid aggressive scaling
- monitor scaling behavior
- combine with Cluster Autoscaler

---

# 🚀 HPA + Cluster Autoscaler

| HPA | Cluster Autoscaler |
|---|---|
| Scales pods | Scales nodes |
| App level | Infrastructure level |

---

# Example

```text
More Traffic
    ↓
HPA Adds Pods
    ↓
No Node Capacity
    ↓
Cluster Autoscaler Adds Nodes
```

---

# 🚨 Common HPA Problems

| Problem | Cause | Solution |
|---|---|---|
| Unknown metrics | Metrics server missing | Install metrics server |
| HPA not scaling | No resource requests | Add CPU requests |
| Pods Pending | Node full | Add nodes |
| Slow scaling | Low metrics frequency | Tune HPA |

---

# 🔥 Troubleshooting Commands

## Describe HPA

```bash
kubectl describe hpa nginx-deployment
```

---

## Metrics Server Logs

```bash
kubectl logs -n kube-system deployment/metrics-server
```

---

## Get Events

```bash
kubectl get events
```

---

## Check Pod Resources

```bash
kubectl describe pod <pod-name>
```

---

# 🔥 Most Asked HPA Interview Questions

# Beginner

## What is HPA?
Automatically scales pods based on metrics.

---

## Why use HPA?
To improve scalability and optimize resources.

---

# Intermediate

## Difference between HPA and Cluster Autoscaler?

| HPA | Cluster Autoscaler |
|---|---|
| Scales pods | Scales nodes |

---

## Why are resource requests required?
HPA calculates scaling using requests.

---

# Advanced

## How does HPA work internally?
Uses metrics server + control loop.

---

## Can HPA scale on custom metrics?
Yes.

---

## What metrics can HPA use?
- CPU
- Memory
- External metrics
- Custom metrics

---

# 🚀 Production Best Practices

- combine HPA + Cluster Autoscaler
- use Prometheus metrics
- define limits/requests
- monitor scaling events
- configure readiness probes

---

# 📚 Best Resources

| Resource | Link |
|---|---|
| Kubernetes Docs | https://kubernetes.io/docs |
| GKE Docs | https://cloud.google.com/kubernetes-engine |
| Metrics Server | https://github.com/kubernetes-sigs/metrics-server |

---

# ⭐ Final Goal

Build:
- Auto Scaling Kubernetes Clusters
- Production GKE Deployments
- High Availability Systems
- Enterprise Kubernetes Platforms
- Cost Optimized Cloud Infrastructure

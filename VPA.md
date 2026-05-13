# 🚀 Vertical Pod Autoscaler (VPA) Mastery Guide
## Beginner → Advanced → Production Kubernetes Auto Scaling Engineer

---

# 📚 What is VPA?

: VPA Vertical Pod Autoscaler (VPA) automatically adjusts:

- CPU requests
- Memory requests

for Kubernetes Pods based on actual resource usage.

---

# 🧠 VPA Architecture

```text
Application Pods
        ↓
Resource Usage Metrics
        ↓
VPA Recommender
        ↓
Updated CPU/Memory Requests
        ↓
Pods Restart with New Resources
```

---

# 🚀 Why Use VPA?

| Benefit | Description |
|---|---|
| Automatic Optimization | Adjusts CPU/RAM automatically |
| Cost Optimization | Reduces overprovisioning |
| Better Performance | Prevents OOMKilled |
| Resource Efficiency | Right-sized containers |
| Simplified Management | Less manual tuning |

---

# 🔥 Difference Between HPA and VPA

| HPA | VPA |
|---|---|
| Scales pod count | Scales pod resources |
| Horizontal scaling | Vertical scaling |
| Adds/removes pods | Adjusts CPU/RAM |
| Best for stateless apps | Best for variable workloads |

---

# 🧠 HPA vs VPA Architecture

```text
HPA
Traffic ↑
   ↓
More Pods

----------------------

VPA
Resource Usage ↑
   ↓
More CPU / Memory
```

---

# 🚀 When to Use VPA?

## Best Use Cases

- databases
- stateful apps
- memory-heavy workloads
- unpredictable resource usage
- backend APIs

---

# 🚫 Avoid VPA With

- aggressive HPA scaling
- fixed resource applications
- latency-sensitive workloads

---

# ☁️ Step 1 — Create GKE Cluster

## Create Cluster

```bash
gcloud container clusters create vpa-cluster \
  --zone us-central1-a \
  --machine-type e2-standard-2 \
  --num-nodes 2
```

---

# 🔗 Step 2 — Connect to Cluster

```bash
gcloud container clusters get-credentials vpa-cluster \
  --zone us-central1-a
```

---

# ✅ Step 3 — Verify Cluster

```bash
kubectl get nodes
```

---

# 🚀 Step 4 — Install Metrics Server

VPA requires metrics collection.

---

# Install Metrics Server

```bash
kubectl apply -f \
https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

# Verify Metrics

```bash
kubectl top nodes
```

---

# 🚀 Step 5 — Install VPA Components

## Clone Autoscaler Repository

```bash
git clone https://github.com/kubernetes/autoscaler.git
```

---

# Move to VPA Directory

```bash
cd autoscaler/vertical-pod-autoscaler
```

---

# Install VPA

```bash
./hack/vpa-up.sh
```

---

# 🧠 What Gets Installed?

| Component | Purpose |
|---|---|
| Recommender | Suggests resources |
| Updater | Updates pods |
| Admission Controller | Applies recommendations |

---

# ✅ Verify VPA Installation

```bash
kubectl get pods -n kube-system
```

---

# Expected Components

```text
vpa-admission-controller
vpa-recommender
vpa-updater
```

---

# 🚀 Step 6 — Create Nginx Deployment

## nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 1

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
              cpu: "100m"
              memory: "128Mi"

            limits:
              cpu: "500m"
              memory: "512Mi"

          ports:
            - containerPort: 80
```

---

# Apply Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

---

# Verify Pods

```bash
kubectl get pods
```

---

# 🚀 Step 7 — Expose Deployment

```bash
kubectl expose deployment nginx-deployment \
  --type=LoadBalancer \
  --port=80 \
  --name=nginx-service
```

---

# Verify Service

```bash
kubectl get svc
```

---

# 🚀 Step 8 — Create VPA Resource

## vpa.yaml

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler

metadata:
  name: nginx-vpa

spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: nginx-deployment

  updatePolicy:
    updateMode: "Auto"
```

---

# 🧠 VPA Modes

| Mode | Description |
|---|---|
| Off | Recommendations only |
| Initial | Set resources at pod creation |
| Auto | Automatically update pods |

---

# Apply VPA

```bash
kubectl apply -f vpa.yaml
```

---

# 🚀 Step 9 — Verify VPA

```bash
kubectl get vpa
```

---

# Describe VPA

```bash
kubectl describe vpa nginx-vpa
```

---

# Expected Output

```text
Recommendation:
  Container Recommendations:
    Target CPU: 200m
    Target Memory: 256Mi
```

---

# 🚀 Step 10 — Generate Load

# Create Load Generator

```bash
kubectl run -it --rm \
  --image=busybox \
  load-generator -- /bin/sh
```

---

# Generate Traffic

```bash
while true; do
  wget -q -O- http://nginx-service;
done
```

---

# 🧠 What Happens?

VPA monitors:
- CPU usage
- memory usage

Then:
```text
Updates recommended resources
```

Pods restart with:
```text
Higher CPU/RAM
```

---

# 🚀 Step 11 — Observe VPA Recommendations

## Check VPA

```bash
kubectl describe vpa nginx-vpa
```

---

# Check Pod Resources

```bash
kubectl describe pod <pod-name>
```

---

# View Metrics

```bash
kubectl top pods
```

---

# 🚀 Step 12 — Understand Pod Restart Behavior

Unlike HPA:
```text
VPA may restart pods
```

because resource requests change.

---

# VPA Workflow

```text
High Resource Usage
        ↓
VPA Recommendation
        ↓
Pod Restart
        ↓
Updated CPU/RAM
```

---

# 🚀 Step 13 — Cleanup Resources

## Delete VPA

```bash
kubectl delete -f vpa.yaml
```

---

# Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

---

# Delete Service

```bash
kubectl delete svc nginx-service
```

---

# Remove VPA Components

```bash
./hack/vpa-down.sh
```

---

# 🔥 Most Important VPA Commands

# Get VPA

```bash
kubectl get vpa
```

---

# Describe VPA

```bash
kubectl describe vpa nginx-vpa
```

---

# Check Metrics

```bash
kubectl top pods
```

---

# Get Pods

```bash
kubectl get pods
```

---

# View Resource Requests

```bash
kubectl describe pod <pod-name>
```

---

# 🚀 Advanced VPA Configuration

## Resource Limits

```yaml
resourcePolicy:
  containerPolicies:
    - containerName: nginx

      minAllowed:
        cpu: 100m
        memory: 128Mi

      maxAllowed:
        cpu: 1
        memory: 1Gi
```

---

# 🚀 VPA Update Modes

# Off Mode

```yaml
updateMode: "Off"
```

Only recommendations.

---

# Initial Mode

```yaml
updateMode: "Initial"
```

Applies at pod creation only.

---

# Auto Mode

```yaml
updateMode: "Auto"
```

Automatically updates resources.

---

# 🚀 VPA + HPA Together

## Important

Using:
```text
HPA + VPA on CPU together
```

can conflict.

---

# Recommended Combination

| Feature | Recommended |
|---|---|
| HPA | CPU scaling |
| VPA | Memory scaling |

---

# 🚀 Real Production Architecture

```text
Users
   ↓
LoadBalancer
   ↓
Service
   ↓
Deployment
   ↓
Pods
   ↓
VPA Monitors Resources
   ↓
Adjust CPU/RAM
```

---

# 🚀 VPA Best Practices

- define resource limits
- monitor recommendations
- avoid frequent restarts
- combine with Cluster Autoscaler
- use for stateful workloads

---

# 🚀 Cluster Autoscaler + VPA

```text
More Resource Needs
        ↓
VPA Increases Pod Resources
        ↓
Node Capacity Full
        ↓
Cluster Autoscaler Adds Nodes
```

---

# 🚨 Common VPA Problems

| Problem | Cause | Solution |
|---|---|---|
| No recommendations | Metrics missing | Install metrics server |
| Frequent pod restarts | Aggressive scaling | Tune update policy |
| Pods Pending | Node full | Add nodes |
| VPA not updating | Wrong targetRef | Fix deployment reference |

---

# 🔥 Troubleshooting Commands

## Check Metrics

```bash
kubectl top pods
```

---

# Describe VPA

```bash
kubectl describe vpa nginx-vpa
```

---

# Check Events

```bash
kubectl get events
```

---

# Verify Metrics Server

```bash
kubectl get apiservices | grep metrics
```

---

# View VPA Pods

```bash
kubectl get pods -n kube-system
```

---

# 🔥 Most Asked VPA Interview Questions

# Beginner

## What is VPA?
Automatically adjusts CPU and memory requests.

---

## Difference between HPA and VPA?

| HPA | VPA |
|---|---|
| Scale pod count | Scale resources |

---

# Intermediate

## Why does VPA restart pods?
Resource requests require pod recreation.

---

## What are VPA components?

| Component | Purpose |
|---|---|
| Recommender | Suggest resources |
| Updater | Restart/update pods |
| Admission Controller | Apply changes |

---

# Advanced

## Can HPA and VPA work together?
Yes, but avoid CPU conflicts.

---

## What metrics does VPA use?
CPU and memory usage.

---

## What is updateMode?
Controls automatic updates.

---

# 🚀 Production Best Practices

- use metrics server
- monitor recommendations
- combine with Cluster Autoscaler
- avoid aggressive autoscaling
- use Prometheus monitoring

---

# 📚 Best Resources

| Resource | Link |
|---|---|
| Kubernetes Docs | https://kubernetes.io/docs |
| VPA GitHub | https://github.com/kubernetes/autoscaler |
| GKE Docs | https://cloud.google.com/kubernetes-engine |

---

# ⭐ Final Goal

Build:
- Self-Optimizing Kubernetes Clusters
- Cost Efficient Infrastructure
- Auto-Healing Platforms
- Enterprise Cloud Systems
- Production Kubernetes Architectures

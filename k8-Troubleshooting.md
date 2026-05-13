# 🚀 Kubernetes Troubleshooting & Debugging Mastery Guide
## Beginner → Advanced → Production Kubernetes Engineer

---

# 📚 What is Kubernetes Troubleshooting?

:Kubernetes troubleshooting is the process of identifying and fixing issues related to:

- Pods
- Deployments
- Services
- Storage
- Networking
- Nodes
- Ingress
- RBAC
- Scaling

:contentReference[oaicite:1]{index=1}

---

# 🧠 Kubernetes Troubleshooting Architecture

```text
Users
   ↓
Ingress / LoadBalancer
   ↓
Service
   ↓
Deployment
   ↓
Pods
   ↓
Node
```

---

# 🚀 Most Common Kubernetes Problems

| Problem | Cause |
|---|---|
| Pods Pending | Insufficient resources |
| CrashLoopBackOff | Application crash |
| ImagePullBackOff | Wrong Docker image |
| Service Not Accessible | Wrong labels/ports |
| PVC Pending | Storage issue |
| Node Not Ready | Node health issue |
| Failed Scheduling | CPU/memory shortage |

---

# 🚀 1. Pods Not Starting

# 📚 Why It Happens

A Pod may fail because of:

- insufficient CPU/memory
- wrong Docker image
- startup failure
- missing environment variables
- invalid YAML

:contentReference[oaicite:2]{index=2}

---

# ✅ Check Pods

```bash
kubectl get pods
```

---

# 🔍 Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# 📄 Check Logs

```bash
kubectl logs <pod-name>
```

---

# 🚨 Common Errors

## Image Errors

```text
ErrImagePull
ImagePullBackOff
```

---

## Resource Errors

```text
0/3 nodes available: insufficient memory
```

:contentReference[oaicite:3]{index=3}

---

# ✅ Fix Image

```bash
kubectl set image deployment/nginx nginx=nginx:latest
```

:contentReference[oaicite:4]{index=4}

---

# ✅ Add Resources

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "200m"

  limits:
    memory: "512Mi"
    cpu: "500m"
```

:contentReference[oaicite:5]{index=5}

---

# Apply Changes

```bash
kubectl apply -f deployment.yaml
```

---

# 🚀 2. Service Not Accessible

# 📚 Why It Happens

- wrong selector labels
- port mismatch
- network policy blocking
- pod not running
- wrong service type

:contentReference[oaicite:6]{index=6}

---

# ✅ Check Services

```bash
kubectl get svc
```

---

# 🔍 Describe Service

```bash
kubectl describe svc <service-name>
```

---

# 🔗 Check Endpoints

```bash
kubectl get endpoints
```

:contentReference[oaicite:7]{index=7}

---

# 🚨 Common Cause

## Service Selector

```yaml
selector:
  app: nginx
```

---

## Pod Label

```yaml
labels:
  app: web
```

No endpoints get created.

:contentReference[oaicite:8]{index=8}

---

# ✅ Fix Labels

Match:
- service selectors
- pod labels

---

# Restart Deployment

```bash
kubectl rollout restart deployment <deployment-name>
```

---

# Test Connectivity

```bash
kubectl port-forward svc/<service-name> 8080:80
```

:contentReference[oaicite:9]{index=9}

---

# 🚀 3. Persistent Volume Claim (PVC) Issues

# 📚 Why It Happens

- storage class missing
- insufficient storage
- access mode mismatch
- PV not bound

:contentReference[oaicite:10]{index=10}

---

# ✅ Check PVC

```bash
kubectl get pvc
```

---

# 🔍 Describe PVC

```bash
kubectl describe pvc <pvc-name>
```

---

# 📦 Check Persistent Volumes

```bash
kubectl get pv
```

:contentReference[oaicite:11]{index=11}

---

# 🚨 Common Error

```text
Pending
```

---

# ✅ Check StorageClass

```bash
kubectl get storageclass
```

---

# Correct Access Mode

```yaml
accessModes:
  - ReadWriteOnce
```

:contentReference[oaicite:12]{index=12}

---

# Apply PVC

```bash
kubectl apply -f pvc.yaml
```

---

# 🚀 4. ConfigMap Key Not Found

# 📚 Why It Happens

Application expects missing ConfigMap key.

:contentReference[oaicite:13]{index=13}

---

# ✅ Get ConfigMaps

```bash
kubectl get configmap
```

---

# 🔍 Describe ConfigMap

```bash
kubectl describe configmap <configmap-name>
```

:contentReference[oaicite:14]{index=14}

---

# 🚨 Common Error

```text
configmap references non-existent key
```

---

# ✅ Fix ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  DB_HOST: mysql
```

:contentReference[oaicite:15]{index=15}

---

# Apply ConfigMap

```bash
kubectl apply -f configmap.yaml
```

---

# Restart Deployment

```bash
kubectl rollout restart deployment <deployment-name>
```

---

# 🚀 5. CrashLoopBackOff

# 📚 Why It Happens

Pod repeatedly crashes due to:

- application crash
- DB connection failure
- startup issue
- liveness probe failure

:contentReference[oaicite:16]{index=16}

---

# 📄 Check Logs

```bash
kubectl logs <pod-name>
```

---

# Previous Logs

```bash
kubectl logs <pod-name> --previous
```

---

# 🔍 Describe Pod

```bash
kubectl describe pod <pod-name>
```

:contentReference[oaicite:17]{index=17}

---

# ✅ Fix Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80

  initialDelaySeconds: 30
  periodSeconds: 10
```

:contentReference[oaicite:18]{index=18}

---

# Restart Deployment

```bash
kubectl rollout restart deployment <deployment-name>
```

---

# 🚀 6. ImagePullBackOff

# 📚 Why It Happens

- wrong image name
- missing tag
- private registry auth failure

:contentReference[oaicite:19]{index=19}

---

# 🔍 Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# ✅ Verify Docker Image

```bash
docker pull nginx:latest
```

:contentReference[oaicite:20]{index=20}

---

# 🔐 Create Docker Registry Secret

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password>
```

:contentReference[oaicite:21]{index=21}

---

# Attach Secret

```yaml
imagePullSecrets:
  - name: regcred
```

:contentReference[oaicite:22]{index=22}

---

# 🚀 7. Node Not Ready

# 📚 Why It Happens

- kubelet stopped
- disk pressure
- memory pressure
- networking issue

:contentReference[oaicite:23]{index=23}

---

# ✅ Check Nodes

```bash
kubectl get nodes
```

---

# 🔍 Describe Node

```bash
kubectl describe node <node-name>
```

:contentReference[oaicite:24]{index=24}

---

# 🚨 Node Conditions

```text
MemoryPressure
DiskPressure
NetworkUnavailable
```

:contentReference[oaicite:25]{index=25}

---

# ✅ Restart Kubelet

```bash
sudo systemctl restart kubelet
```

---

# Free Disk Space

```bash
docker system prune -a
```

:contentReference[oaicite:26]{index=26}

---

# 🚀 8. Failed Scheduling

# 📚 Why It Happens

- insufficient resources
- node taints
- affinity rules
- selector mismatch

:contentReference[oaicite:27]{index=27}

---

# 🔍 Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# 🚨 Common Error

```text
0/3 nodes available
```

:contentReference[oaicite:28]{index=28}

---

# ✅ Check Resource Usage

```bash
kubectl top nodes
```

---

# Remove Taint

```bash
kubectl taint nodes <node-name> key=value:NoSchedule-
```

---

# Scale Deployment

```bash
kubectl scale deployment nginx --replicas=5
```

:contentReference[oaicite:29]{index=29}

---

# 🚀 9. RBAC Authorization Errors

# 📚 Why It Happens

- missing role bindings
- incorrect permissions
- service account restrictions

:contentReference[oaicite:30]{index=30}

---

# 🚨 Common Error

```text
Forbidden
User cannot list pods
```

:contentReference[oaicite:31]{index=31}

---

# ✅ Check Permissions

```bash
kubectl auth can-i list pods
```

---

# Check Roles

```bash
kubectl get roles
kubectl get rolebindings
```

:contentReference[oaicite:32]{index=32}

---

# ✅ Create RoleBinding

```bash
kubectl create rolebinding dev-binding \
  --clusterrole=edit \
  --serviceaccount=default:default
```

:contentReference[oaicite:33]{index=33}

---

# 🚀 10. Ingress Controller Issues

# 📚 Why It Happens

- bad ingress rules
- TLS issue
- backend unavailable
- DNS not configured

:contentReference[oaicite:34]{index=34}

---

# ✅ Check Ingress

```bash
kubectl get ingress
```

---

# 🔍 Describe Ingress

```bash
kubectl describe ingress <ingress-name>
```

---

# Verify Controller

```bash
kubectl get pods -n ingress-nginx
```

:contentReference[oaicite:35]{index=35}

---

# 🔐 Create TLS Secret

```bash
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

:contentReference[oaicite:36]{index=36}

---

# Apply Ingress

```bash
kubectl apply -f ingress.yaml
```

---

# 🔥 Most Important Kubernetes Debug Commands

# Cluster Commands

```bash
kubectl cluster-info
kubectl get nodes
kubectl top nodes
```

:contentReference[oaicite:37]{index=37}

---

# Pod Commands

```bash
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

:contentReference[oaicite:38]{index=38}

---

# Service Commands

```bash
kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints
```

:contentReference[oaicite:39]{index=39}

---

# Storage Commands

```bash
kubectl get pv
kubectl get pvc
kubectl describe pvc <pvc-name>
```

:contentReference[oaicite:40]{index=40}

---

# 🚀 Kubernetes Troubleshooting Flow

```text
Pod Issue
   ↓
kubectl get pods
   ↓
kubectl describe pod
   ↓
kubectl logs
   ↓
Check Events
   ↓
Fix YAML / Resources / Image / Network
   ↓
kubectl apply -f
```

:contentReference[oaicite:41]{index=41}

---

# 🧠 Real Production Kubernetes Architecture

```text
Users
   ↓
LoadBalancer
   ↓
Ingress
   ↓
Service
   ↓
Deployment
   ↓
Pods
   ↓
Node
```

---

# 🚀 Production Best Practices

- use readiness/liveness probes
- define resource requests/limits
- enable monitoring
- centralize logs
- secure RBAC
- use namespaces
- implement autoscaling

---

# 🔥 Most Asked Kubernetes Interview Questions

# Beginner

## What is a Pod?
Smallest deployable unit in Kubernetes.

---

## Difference between Deployment and Pod?

| Pod | Deployment |
|---|---|
| Single instance | Manages pods |
| Manual | Self-healing |

---

# Intermediate

## What is CrashLoopBackOff?
Pod repeatedly crashing.

---

## What is ConfigMap?
Stores configuration data.

---

## What is PVC?
Persistent storage request.

---

# Advanced

## Difference between HPA and Cluster Autoscaler?

| HPA | Cluster Autoscaler |
|---|---|
| Scales pods | Scales nodes |

---

## How does Kubernetes scheduling work?
Scheduler assigns pods based on resources/constraints.

---

## Explain Ingress.
Manages external HTTP/HTTPS routing.

---

# 📚 Best Resources

| Resource | Link |
|---|---|
| Kubernetes Docs | https://kubernetes.io/docs |
| GKE Docs | https://cloud.google.com/kubernetes-engine |
| kubectl Cheat Sheet | https://kubernetes.io/docs/reference/kubectl/cheatsheet |

---

# ⭐ Final Goal

Build:
- Production Kubernetes Clusters
- Scalable Cloud Platforms
- Auto-Healing Systems
- Enterprise Infrastructure
- High Availability Architectures
- DevOps & Platform Engineering Systems

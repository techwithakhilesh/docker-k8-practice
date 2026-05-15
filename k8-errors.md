# 🚨 Kubernetes Production Errors Cheat Sheet

Production-ready Kubernetes troubleshooting guide for Pods, Nodes, Networking, HPA, RBAC, and Images.

---

# 🔴 CrashLoopBackOff

## 📌 Component
POD

## 📌 Meaning
Container repeatedly crashes. Kubernetes keeps restarting it with increasing back-off delays.

---

## 🔍 Diagnose

```bash
kubectl logs <pod-name> -n <namespace> --previous

kubectl describe pod <pod-name> -n <namespace>
```

---

## ✅ Fix

```bash
# Check previous crash logs
kubectl logs <pod-name> -n <namespace> --previous

# Fix app issue then restart deployment
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

---

# 🔴 OOMKilled

## 📌 Component
POD

## 📌 Meaning
Container exceeded memory limit and was killed by Linux OOM handler.

---

## 🔍 Diagnose

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Reason: OOMKilled
```

---

## ✅ Fix

```yaml
resources:
  limits:
    memory: "512Mi"

  requests:
    memory: "256Mi"
```

---

# 🔴 Pending (No Resources)

## 📌 Component
POD

## 📌 Meaning
No node has enough CPU or memory to schedule the pod.

---

## 🔍 Diagnose

```bash
kubectl describe pod <pod-name> -n <namespace>

kubectl get nodes -o wide
```

---

## ✅ Fix

```bash
# Scale node pool
gcloud container clusters resize <cluster-name> \
  --num-nodes 5 \
  --region <region>
```

OR reduce deployment resource requests.

---

# 🔴 Pending (No Taints Match)

## 📌 Component
POD

## 📌 Meaning
Pod tolerations or nodeSelector do not match available nodes.

---

## 🔍 Diagnose

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
didn't match node selector
```

---

## ✅ Fix

```yaml
nodeSelector:
  cloud.google.com/gke-nodepool: pool-name
```

Fix or remove invalid nodeSelector.

---

# 🔴 Evicted

## 📌 Component
POD

## 📌 Meaning
Node was under pressure and kubelet evicted low-priority pods.

---

## 🔍 Diagnose

```bash
kubectl get pods -n <namespace> | grep Evicted

kubectl describe pod <pod-name> -n <namespace>
```

---

## ✅ Fix

```bash
kubectl delete pod <pod-name> -n <namespace>
```

### Long-Term Fix

- Add PodDisruptionBudget
- Increase node resources

---

# 🔴 Terminating (Stuck)

## 📌 Component
POD

## 📌 Meaning
Pod stuck in terminating state.

---

## 🔍 Diagnose

```bash
kubectl get pods -n <namespace>

kubectl describe pod <pod-name> -n <namespace>
```

---

## ✅ Fix

```bash
kubectl delete pod <pod-name> \
  --grace-period=0 \
  --force \
  -n <namespace>
```

---

# 🔴 CreateContainerConfigError

## 📌 Component
POD

## 📌 Meaning
Missing ConfigMap, Secret, or invalid environment variable configuration.

---

## 🔍 Diagnose

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Error: secret not found
```

---

## ✅ Fix

```bash
kubectl get secret -n <namespace>

kubectl get configmap -n <namespace>
```

Create missing Secret or ConfigMap then redeploy.

---

# 🔴 RunContainerError

## 📌 Component
POD

## 📌 Meaning
Container starts but immediately fails.

---

## 🔍 Diagnose

```bash
kubectl logs <pod-name> -n <namespace>

kubectl describe pod <pod-name> -n <namespace>
```

---

## ✅ Fix

```bash
kubectl run debug \
  --image=<image-name> \
  -it --rm -- /bin/sh
```

### Check

- Entrypoint
- Permissions
- Startup scripts

---

# 🔴 ImagePullBackOff

## 📌 Component
IMAGE

## 📌 Meaning
Kubernetes cannot pull image from registry.

---

## 🔍 Diagnose

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Back-off pulling image
```

---

## ✅ Fix

```bash
docker pull <image-name>:<tag>
```

### For Private Registry

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password> \
  -n <namespace>
```

---

# 🔴 ErrImagePull

## 📌 Component
IMAGE

## 📌 Meaning
Initial image pull failed.

---

## 🔍 Diagnose

```bash
kubectl describe pod <pod-name> -n <namespace>
```

---

## ✅ Fix

```bash
gcloud artifacts repositories list
```

Grant Artifact Registry permission:

```bash
gcloud projects add-iam-policy-binding <project-id> \
  --member=serviceAccount:<service-account> \
  --role=roles/artifactregistry.reader
```

---

# 🔴 Node NotReady

## 📌 Component
NODE

## 📌 Meaning
Node unhealthy and cannot accept pods.

---

## 🔍 Diagnose

```bash
kubectl get nodes

kubectl describe node <node-name>

kubectl get events \
  --field-selector involvedObject.name=<node-name>
```

---

## ✅ Fix

```bash
kubectl drain <node-name> --ignore-daemonsets
```

Restart kubelet:

```bash
sudo systemctl restart kubelet
```

Or replace node:

```bash
kubectl delete node <node-name>
```

---

# 🔴 DiskPressure

## 📌 Component
NODE

## 📌 Meaning
Node running out of disk space.

---

## 🔍 Diagnose

```bash
kubectl describe node <node-name>
```

Look for:

```text
Condition: DiskPressure
```

---

## ✅ Fix

```bash
docker system prune -af
```

### Long-Term Fix

- Increase node disk size
- Use larger boot disks

---

# 🔴 MemoryPressure

## 📌 Component
NODE

## 📌 Meaning
Node low on memory.

---

## 🔍 Diagnose

```bash
kubectl top nodes

kubectl describe node <node-name>
```

---

## ✅ Fix

```bash
kubectl top pods -n <namespace> --sort-by=memory
```

Then:

- Add more nodes
- Reduce pod memory usage

---

# 🔴 Service Unreachable

## 📌 Component
NETWORK

## 📌 Meaning
Pod cannot connect to service.

---

## 🔍 Diagnose

```bash
kubectl run dbg \
  --image=busybox \
  -it --rm -- \
  nslookup <service-name>.<namespace>.svc.cluster.local
```

---

## ✅ Fix

```bash
kubectl get svc -n <namespace>

kubectl describe svc <service-name> -n <namespace>

kubectl get pods -n <namespace> --show-labels
```

---

# 🔴 DNS Resolution Failure

## 📌 Component
NETWORK

## 📌 Meaning
CoreDNS not resolving service names.

---

## 🔍 Diagnose

```bash
kubectl get pods -n kube-system | grep coredns

kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## ✅ Fix

```bash
kubectl rollout restart deployment/coredns -n kube-system
```

Check config:

```bash
kubectl get configmap coredns \
  -n kube-system -o yaml
```

---

# 🔴 Connection Refused / Timeout

## 📌 Component
NETWORK

## 📌 Meaning
Service selector mismatch or no ready pods.

---

## 🔍 Diagnose

```bash
kubectl describe svc <service-name> -n <namespace>

kubectl get endpoints -n <namespace>
```

---

## ✅ Fix

```bash
kubectl label pod <pod-name> app=<label> -n <namespace>
```

Check readiness:

```bash
kubectl get pods -n <namespace> -o wide
```

---

# 🔴 HPA Not Scaling

## 📌 Component
HPA

## 📌 Meaning
HPA stuck at minimum replicas.

---

## 🔍 Diagnose

```bash
kubectl describe hpa -n <namespace>

kubectl get apiservices | grep metrics
```

---

## ✅ Fix

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify:

```bash
kubectl top pods -n <namespace>
```

---

# 🔴 FailedGetScale

## 📌 Component
HPA

## 📌 Meaning
HPA unable to read metrics.

---

## 🔍 Diagnose

```bash
kubectl describe hpa -n <namespace>
```

Look for:

```text
FailedGetScale
```

---

## ✅ Fix

```bash
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods
```

Restart metrics-server:

```bash
kubectl rollout restart deployment/metrics-server \
  -n kube-system
```

---

# 🔴 Forbidden 403

## 📌 Component
RBAC

## 📌 Meaning
User or ServiceAccount lacks permissions.

---

## 🔍 Diagnose

```bash
kubectl auth can-i <verb> <resource> \
  --as=system:serviceaccount:<namespace>:<service-account>

kubectl describe rolebinding -n <namespace>
```

---

## ✅ Fix

```bash
kubectl create rolebinding <binding-name> \
  --clusterrole=<role> \
  --serviceaccount=<namespace>:<service-account> \
  -n <namespace>
```

---

# 🔴 Unauthorized 401

## 📌 Component
RBAC

## 📌 Meaning
Credentials expired or kubeconfig invalid.

---

## 🔍 Diagnose

```bash
kubectl config view

gcloud container clusters get-credentials <cluster-name> \
  --region <region>
```

---

## ✅ Fix

```bash
gcloud container clusters get-credentials <cluster-name> \
  --region <region>
```

Verify:

```bash
kubectl get nodes
```

---

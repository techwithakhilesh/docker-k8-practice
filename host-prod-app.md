# Practical GKE Production Setup Guide 🚀

Complete Production-Grade Microservices Deployment on Google Kubernetes Engine (GKE)

---

# 📌 Technologies Covered

- Kubernetes
- Docker
- Google Kubernetes Engine (GKE)
- Artifact Registry
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Prometheus
- Grafana
- NGINX Ingress
- TLS / HTTPS
- Cloud DNS

---

# 📌 Step 1 — Prerequisites & Project Setup

## Install Required Tools

```bash
# Install kubectl
gcloud components install kubectl

# Install GKE auth plugin
gcloud components install gke-gcloud-auth-plugin
```

---

## Set Environment Variables

```bash
# GCP Project ID
export PROJECT_ID="my-production-project"

# GCP Region
export REGION="us-central1"

# GKE Cluster Name
export CLUSTER_NAME="prod-cluster"
```

---

## Configure GCP Project & Region

```bash
# Set active GCP project
gcloud config set project $PROJECT_ID

# Set default compute region
gcloud config set compute/region $REGION
```

---

## Enable Required APIs

```bash
gcloud services enable \
  container.googleapis.com \
  artifactregistry.googleapis.com \
  dns.googleapis.com \
  monitoring.googleapis.com \
  cloudresourcemanager.googleapis.com
```

---

# 📌 Step 2 — Create GKE Cluster

## Create Production GKE Cluster

```bash
gcloud container clusters create $CLUSTER_NAME \
  --region $REGION \
  --num-nodes 3 \
  --min-nodes 2 \
  --max-nodes 10 \
  --enable-autoscaling \
  --machine-type e2-standard-4 \
  --enable-autorepair \
  --enable-autoupgrade \
  --enable-ip-alias \
  --workload-pool="${PROJECT_ID}.svc.id.goog" \
  --enable-shielded-nodes \
  --release-channel regular
```

---

## Connect kubectl to Cluster

```bash
gcloud container clusters get-credentials $CLUSTER_NAME --region $REGION
```

---

## Verify Nodes

```bash
kubectl get nodes
```

---

# 📌 Step 3 — Docker Container & Artifact Registry

## Create Artifact Registry Repository

```bash
gcloud artifacts repositories create prod-repo \
  --repository-format docker \
  --location $REGION \
  --description "Production container images"
```

---

## Configure Docker Authentication

```bash
gcloud auth configure-docker ${REGION}-docker.pkg.dev
```

---

## Define Image Base Path

```bash
export IMAGE_BASE="${REGION}-docker.pkg.dev/${PROJECT_ID}/prod-repo"
```

---

# 📌 Production Dockerfile Example

Save this as `Dockerfile` inside each microservice.

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production

COPY . .

FROM node:20-alpine

WORKDIR /app

COPY --from=builder /app .

EXPOSE 8080

USER node

CMD ["node", "server.js"]
```

---

## Build Docker Images

```bash
docker build -t ${IMAGE_BASE}/service-a:v1.0.0 ./service-a

docker build -t ${IMAGE_BASE}/service-b:v1.0.0 ./service-b
```

---

## Push Images to Artifact Registry

```bash
docker push ${IMAGE_BASE}/service-a:v1.0.0

docker push ${IMAGE_BASE}/service-b:v1.0.0
```

---

# 📌 Step 4 — Deploy Microservices to Kubernetes

## service-a-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: service-a
  namespace: production

spec:
  replicas: 3

  selector:
    matchLabels:
      app: service-a

  template:
    metadata:
      labels:
        app: service-a

    spec:
      containers:
        - name: service-a

          image: us-central1-docker.pkg.dev/PROJECT_ID/prod-repo/service-a:v1.0.0

          ports:
            - containerPort: 8080

          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"

            limits:
              memory: "512Mi"
              cpu: "1000m"

          readinessProbe:
            httpGet:
              path: /healthz
              port: 8080

            initialDelaySeconds: 10
            periodSeconds: 5

          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080

            initialDelaySeconds: 30
            periodSeconds: 10

          env:
            - name: NODE_ENV
              value: "production"

---
apiVersion: v1
kind: Service

metadata:
  name: service-a
  namespace: production

spec:
  selector:
    app: service-a

  ports:
    - port: 80
      targetPort: 8080

  type: ClusterIP
```

---

## Create Namespace

```bash
kubectl create namespace production
```

---

## Deploy Applications

```bash
kubectl apply -f service-a-deployment.yaml

kubectl apply -f service-b-deployment.yaml
```

---

## Verify Pods

```bash
kubectl get pods -n production
```

---

# 📌 Step 5 — Horizontal Pod Autoscaler (HPA)

## hpa-service-a.yaml

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: service-a-hpa
  namespace: production

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: service-a

  minReplicas: 2
  maxReplicas: 20

  metrics:
    - type: Resource
      resource:
        name: cpu

        target:
          type: Utilization
          averageUtilization: 70

    - type: Resource
      resource:
        name: memory

        target:
          type: Utilization
          averageUtilization: 80

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60

      policies:
        - type: Pods
          value: 4
          periodSeconds: 60

    scaleDown:
      stabilizationWindowSeconds: 300
```

---

## Apply HPA

```bash
kubectl apply -f hpa-service-a.yaml
```

---

## Verify HPA

```bash
kubectl get hpa -n production
```

---

## Describe HPA

```bash
kubectl describe hpa service-a-hpa -n production
```

---

# 📌 Step 6 — Vertical Pod Autoscaler (VPA)

## Install VPA

```bash
git clone https://github.com/kubernetes/autoscaler.git

cd autoscaler/vertical-pod-autoscaler

./hack/vpa-up.sh
```

---

## vpa-service-a.yaml

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler

metadata:
  name: service-a-vpa
  namespace: production

spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: service-a

  updatePolicy:
    updateMode: "Auto"

  resourcePolicy:
    containerPolicies:
      - containerName: service-a

        minAllowed:
          cpu: "100m"
          memory: "64Mi"

        maxAllowed:
          cpu: "4"
          memory: "4Gi"

        controlledResources:
          - "cpu"
          - "memory"
```

---

## Apply VPA

```bash
kubectl apply -f vpa-service-a.yaml
```

---

## Verify VPA

```bash
kubectl get vpa -n production
```

---

## Check VPA Recommendations

```bash
kubectl describe vpa service-a-vpa -n production
```

---

# 📌 Step 7 — Install Prometheus & Grafana

## Add Helm Repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo add grafana https://grafana.github.io/helm-charts

helm repo update
```

---

## Install kube-prometheus-stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi \
  --set grafana.adminPassword="your-secure-password" \
  --set grafana.persistence.enabled=true \
  --set grafana.persistence.size=10Gi \
  --set alertmanager.enabled=true
```

---

## Verify Monitoring Pods

```bash
kubectl get pods -n monitoring
```

---

# 📌 Prometheus Scrape Annotations

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

---

## Access Grafana Locally

```bash
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
```

Open:

```text
http://localhost:3000
```

---

# 📌 Step 8 — Ingress + TLS + Domain Setup

## Install NGINX Ingress Controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer
```

---

## Get External IP

```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

---

# 📌 Install cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

---

## Verify cert-manager Pods

```bash
kubectl get pods -n cert-manager
```

---

# 📌 Best Practices

✅ Use namespaces  
✅ Use readiness/liveness probes  
✅ Use autoscaling  
✅ Enable HTTPS  
✅ Use monitoring & alerts  
✅ Use persistent storage  
✅ Use private container registry  
✅ Enable auto-upgrade & auto-repair  
✅ Set resource requests & limits  
✅ Use production-grade ingress

---

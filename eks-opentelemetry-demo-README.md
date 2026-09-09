# 🚀 OpenTelemetry Demo on Amazon EKS

## 📌 Project Overview

This project demonstrates deploying the **OpenTelemetry Demo** on an **Amazon EKS cluster** using `eksctl`, Kubernetes, and Helm.

The environment includes a Kubernetes cluster with private managed worker nodes and an OpenTelemetry-based observability stack.

### Technologies Used

- AWS EKS
- AWS CLI
- eksctl
- kubectl
- Kubernetes
- Helm
- Docker
- OpenTelemetry
- Prometheus
- Grafana
- Jaeger
- OpenTelemetry Collector

---

# ☁️ Create EKS Cluster

## Prerequisites

Install and configure the following tools:

- **AWS CLI**
- **eksctl**
- **kubectl**
- **Helm**

Configure AWS CLI:

```bash
aws configure
```

Verify AWS access:

```bash
aws sts get-caller-identity
```

Verify the tools:

```bash
aws --version
eksctl version
kubectl version --client
helm version
```

---

# 1. Create EKS Control Plane

Create the EKS cluster without a node group:

```bash
eksctl create cluster \
  --name=observability \
  --region=us-east-1 \
  --zones=us-east-1a,us-east-1b \
  --without-nodegroup
```

Verify the cluster:

```bash
eksctl get cluster --region us-east-1
```

Check the EKS cluster status:

```bash
aws eks describe-cluster \
  --name observability \
  --region us-east-1 \
  --query 'cluster.status'
```

Expected:

```text
"ACTIVE"
```

---

# 2. Associate IAM OIDC Provider

Associate an IAM OIDC provider with the EKS cluster:

```bash
eksctl utils associate-iam-oidc-provider \
  --region us-east-1 \
  --cluster observability \
  --approve
```

The OIDC provider is useful for IAM Roles for Service Accounts (IRSA), allowing Kubernetes workloads to securely access AWS services without using static AWS credentials.

---

# 3. Create Managed Private Node Group

Create the managed worker node group:

```bash
eksctl create nodegroup \
  --cluster=observability \
  --region=us-east-1 \
  --name=observability-ng-private \
  --node-type=t3.medium \
  --nodes-min=2 \
  --nodes-max=3 \
  --node-volume-size=20 \
  --managed \
  --asg-access \
  --external-dns-access \
  --full-ecr-access \
  --appmesh-access \
  --alb-ingress-access \
  --node-private-networking
```

Verify the node group:

```bash
eksctl get nodegroup \
  --cluster=observability \
  --region=us-east-1
```

Check Kubernetes nodes:

```bash
kubectl get nodes
```

Expected:

```text
NAME                            STATUS   ROLES
ip-xxx-xxx-xxx-xxx...           Ready    <none>
ip-xxx-xxx-xxx-xxx...           Ready    <none>
```

---

# 4. Update kubeconfig

Configure `kubectl` to communicate with the EKS cluster:

```bash
aws eks update-kubeconfig \
  --name observability \
  --region us-east-1
```

Verify the current context:

```bash
kubectl config current-context
```

Check cluster access:

```bash
kubectl get nodes
```

---

# 🔭 Install OpenTelemetry Demo

## 5. Add OpenTelemetry Helm Repository

Add the OpenTelemetry Helm repository:

```bash
helm repo add open-telemetry \
  https://open-telemetry.github.io/opentelemetry-helm-charts
```

Update the repository:

```bash
helm repo update
```

Search for the demo chart:

```bash
helm search repo open-telemetry/opentelemetry-demo
```

---

# 6. Install OpenTelemetry Demo

Install the OpenTelemetry Demo into the `default` namespace:

```bash
helm install my-otel-demo \
  open-telemetry/opentelemetry-demo \
  --namespace default
```

Verify the Helm release:

```bash
helm list -n default
```

Expected:

```text
NAME           NAMESPACE   STATUS
my-otel-demo   default     deployed
```

---

# 7. Verify OpenTelemetry Pods

Check all pods:

```bash
kubectl get pods -n default
```

Watch pods while they start:

```bash
kubectl get pods -n default -w
```

Check services:

```bash
kubectl get svc -n default
```

Check deployments:

```bash
kubectl get deployments -n default
```

Check endpoints:

```bash
kubectl get endpoints -n default
```

---

# 8. Access the OpenTelemetry Demo

The OpenTelemetry Demo frontend is exposed through the `frontend-proxy` service.

Check the service:

```bash
kubectl get svc frontend-proxy -n default
```

Check its ports:

```bash
kubectl describe svc frontend-proxy -n default
```

For local testing, port-forward the frontend:

```bash
kubectl port-forward \
  svc/frontend-proxy \
  8080:8080 \
  -n default
```

Open:

```text
http://localhost:8080
```

If local port `8080` is already in use, use another local port:

```bash
kubectl port-forward \
  svc/frontend-proxy \
  8081:8080 \
  -n default
```

Then open:

```text
http://localhost:8081
```

---

# 📊 OpenTelemetry Observability Components

The OpenTelemetry Demo provides an application environment that can be observed through multiple telemetry signals.

## Prometheus

Prometheus collects and stores application metrics.

Check the Prometheus pod:

```bash
kubectl get pods -n default | grep prometheus
```

Check the service:

```bash
kubectl get svc -n default | grep prometheus
```

---

## Grafana

Grafana is used to visualize metrics and observability data.

Check Grafana:

```bash
kubectl get pods -n default | grep grafana
```

Check the Grafana service:

```bash
kubectl get svc grafana -n default
```

If the Grafana Service exposes port `80`, access it with:

```bash
kubectl port-forward svc/grafana 3000:80 -n default
```

Then open:

```text
http://localhost:3000/grafana/
```

---

## Jaeger

Jaeger is used for distributed tracing.

Check Jaeger:

```bash
kubectl get pods -n default | grep jaeger
```

Check the services:

```bash
kubectl get svc -n default | grep jaeger
```

The Jaeger UI can also be accessed through the OpenTelemetry Demo frontend proxy when that route is available:

```text
http://localhost:8080/jaeger/ui/
```

Or, when using port `8081`:

```text
http://localhost:8081/jaeger/ui/
```

---

# 🔎 Useful Kubernetes Commands

### List all resources

```bash
kubectl get all -n default
```

### List pods with IP addresses

```bash
kubectl get pods -n default -o wide
```

### Describe a pod

```bash
kubectl describe pod <pod-name> -n default
```

### View pod logs

```bash
kubectl logs <pod-name> -n default
```

### View previous container logs

Useful for `CrashLoopBackOff`:

```bash
kubectl logs <pod-name> -n default --previous
```

### Check pod events

```bash
kubectl get events -n default --sort-by=.lastTimestamp
```

### Check service endpoints

```bash
kubectl get endpoints -n default
```

---

# 🔧 Helm Commands

Check the release:

```bash
helm status my-otel-demo -n default
```

View Helm values:

```bash
helm get values my-otel-demo -n default
```

View installed Kubernetes manifests:

```bash
helm get manifest my-otel-demo -n default
```

Render the chart without installing:

```bash
helm template opentelemetry-demo \
  open-telemetry/opentelemetry-demo \
  --namespace default \
  > opentelemetry-demo.yaml
```

> `helm template` only generates Kubernetes YAML. It does not deploy the application.

---

# 🛠️ Troubleshooting

## Check Pending Pods

```bash
kubectl get pods -n default
```

If a pod is `Pending`:

```bash
kubectl describe pod <pod-name> -n default
```

Look at the `Events` section for scheduling or resource errors.

## CrashLoopBackOff

```bash
kubectl logs <pod-name> -n default --previous
```

Then:

```bash
kubectl describe pod <pod-name> -n default
```

## OOMKilled

If a container shows `OOMKilled`, check:

```bash
kubectl describe pod <pod-name> -n default
```

Check node resources:

```bash
kubectl top nodes
```

Check pod resource usage:

```bash
kubectl top pods -n default
```

> `kubectl top` requires Metrics Server to be available.

---

# 🧹 Cleanup

## Uninstall OpenTelemetry Demo

```bash
helm uninstall my-otel-demo -n default
```

Verify:

```bash
helm list -A
```

## Delete the EKS Cluster

When the project is complete and you no longer need the AWS resources:

```bash
eksctl delete cluster \
  --name observability \
  --region us-east-1
```

> **Important:** EKS resources can incur AWS charges. Delete the cluster and associated resources when you are finished with the lab.

---

# 🎯 Project Workflow

```text
AWS CLI
   │
   ▼
eksctl
   │
   ▼
Amazon EKS
   │
   ├── Control Plane
   │
   └── Managed Private Node Group
          │
          ▼
       Kubernetes
          │
          ▼
         Helm
          │
          ▼
   OpenTelemetry Demo
          │
     ┌────┼───────────────┐
     ▼    ▼               ▼
 Metrics  Traces         Logs
     │    │               │
     ▼    ▼               ▼
Prometheus Jaeger    OpenTelemetry
     │                    Collector
     ▼
  Grafana
```

---

# 📚 What I Practiced

Through this project, I practiced:

- Amazon EKS cluster provisioning
- `eksctl`
- AWS CLI configuration
- Kubernetes cluster management
- Managed EKS node groups
- Private worker nodes
- IAM OIDC provider
- Helm chart deployment
- OpenTelemetry
- Prometheus
- Grafana
- Jaeger
- Distributed tracing
- Kubernetes Services
- Port forwarding
- Kubernetes troubleshooting
- Container resource management
- Observability in a microservices environment

---

# ✅ Project Result

Successfully deployed the **OpenTelemetry Demo on Amazon EKS** using `eksctl` and Helm, with Kubernetes-based application services and an observability stack including **OpenTelemetry, Prometheus, Grafana, and Jaeger**.


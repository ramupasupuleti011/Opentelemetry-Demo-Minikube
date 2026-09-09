# OpenTelemetry Demo on Minikube

## 📌 Project Overview

This project demonstrates a local **OpenTelemetry observability environment** running on **Kubernetes with Minikube**.

The OpenTelemetry Demo is deployed using **Helm** and includes a microservices-based application with observability components such as:

- OpenTelemetry Collector
- Prometheus
- Grafana
- Jaeger
- OpenTelemetry Demo microservices
- Frontend and Frontend Proxy
- Kafka and supporting services

The entire demo was deployed locally using **Minikube with the Docker driver**.

---

## 🏗️ Architecture

```text
                        Local MacBook
                             │
                             ▼
                       Docker Desktop
                             │
                             ▼
                         Minikube
                             │
                 ┌───────────┴───────────┐
                 │                       │
                 ▼                       ▼
        OpenTelemetry Demo        Observability Stack
                 │                       │
       ┌─────────┴─────────┐       ┌─────┴─────┐
       │                   │       │           │
    Frontend          Microservices Prometheus Grafana
       │                   │                   │
       ▼                   ▼                   ▼
Frontend Proxy      Checkout / Cart /      Metrics
                    Payment / Shipping

                         Jaeger
                           │
                           ▼
                         Traces
```

---

# 🚀 Installation

## Prerequisites

Install the following tools before starting:

- Docker Desktop
- Minikube
- kubectl
- Helm

Verify installations:

```bash
docker --version
minikube version
kubectl version --client
helm version
```

---

# 1. Start Minikube

Start a clean Minikube cluster using the Docker driver.

For the OpenTelemetry Demo, allocate sufficient CPU and memory:

```bash
minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=10240
```

Check Minikube:

```bash
minikube status
```

Check Kubernetes nodes:

```bash
kubectl get nodes
```

Expected:

```text
NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane   ...
```

Check the current Kubernetes context:

```bash
kubectl config current-context
```

Expected:

```text
minikube
```

---

# 2. Add OpenTelemetry Helm Repository

Add the official OpenTelemetry Helm repository:

```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
```

Update Helm repositories:

```bash
helm repo update
```

Verify the chart:

```bash
helm search repo open-telemetry/opentelemetry-demo
```

---

# 3. Install OpenTelemetry Demo

Install the OpenTelemetry Demo using Helm:

```bash
helm install my-otel-demo \
  open-telemetry/opentelemetry-demo \
  -n default
```

Check the Helm release:

```bash
helm list -n default
```

Expected status:

```text
STATUS: deployed
```

---

# 4. Verify Kubernetes Resources

Check all pods:

```bash
kubectl get pods -n default
```

Watch pods while they are starting:

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

---

# 5. Access the OpenTelemetry Demo

The demo frontend is exposed through the `frontend-proxy` service.

Port-forward the service:

```bash
kubectl port-forward svc/frontend-proxy 8081:8080 -n default
```

Open the application:

```text
http://localhost:8081
```

> **Note:** Port `8080` may already be used by another local application. In that case, `8081:8080` maps local port `8081` to the Kubernetes service port `8080`.

---

# 6. Access Grafana

Check the Grafana service:

```bash
kubectl get svc grafana -n default
```

In this demo, the Grafana Kubernetes Service exposes port `80`.

Port-forward Grafana:

```bash
kubectl port-forward svc/grafana 3000:80 -n default
```

Then access:

```text
http://localhost:3000/grafana/
```

If local port `3000` is already occupied, use another local port:

```bash
kubectl port-forward svc/grafana 3001:80 -n default
```

However, because the demo's Grafana configuration may redirect to its configured `/grafana/` base URL, using the configured local port is recommended.

---

# 7. Access Jaeger

Check the Jaeger service:

```bash
kubectl get svc -n default | grep jaeger
```

You can also inspect the service ports:

```bash
kubectl describe svc jaeger -n default
```

Port-forward the appropriate Jaeger service port and open the Jaeger UI in your browser.

---

# 8. Useful Kubernetes Commands

### Check pods

```bash
kubectl get pods
```

### Check pods with IP addresses

```bash
kubectl get pods -o wide
```

### Check services

```bash
kubectl get svc
```

### Describe a pod

```bash
kubectl describe pod <pod-name>
```

### View logs

```bash
kubectl logs <pod-name>
```

### Follow logs

```bash
kubectl logs -f <pod-name>
```

### Check endpoints

```bash
kubectl get endpoints
```

### Check deployments

```bash
kubectl get deployments
```

---

# 9. Helm Commands

List Helm releases:

```bash
helm list -A
```

Check the OpenTelemetry Demo release:

```bash
helm status my-otel-demo -n default
```

View Helm values:

```bash
helm get values my-otel-demo -n default
```

View generated Kubernetes resources:

```bash
helm get manifest my-otel-demo -n default
```

> `helm template` only renders Kubernetes YAML locally. It does **not** install or deploy the application.

If you want to render the chart into a YAML file:

```bash
helm template opentelemetry-demo \
  open-telemetry/opentelemetry-demo \
  --namespace default \
  > opentelemetry-demo.yaml
```

---

# 10. Troubleshooting

## Check for Pending pods

```bash
kubectl get pods -n default
```

If pods are `Pending`, check:

```bash
kubectl describe pod <pod-name> -n default
```

## Check for CrashLoopBackOff

```bash
kubectl logs <pod-name> -n default
```

For logs from the previous crashed container:

```bash
kubectl logs <pod-name> -n default --previous
```

## Check for OOMKilled

```bash
kubectl describe pod <pod-name> -n default
```

If a container is `OOMKilled`, the Minikube environment may need more memory.

Example:

```bash
minikube stop
minikube delete

minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=10240
```

---

# 11. Clean Up

Uninstall the OpenTelemetry Demo:

```bash
helm uninstall my-otel-demo -n default
```

Delete the entire Minikube cluster:

```bash
minikube delete
```

Check Minikube:

```bash
minikube status
```

---

# 🧪 What I Practiced

Through this project, I practiced:

- Kubernetes fundamentals
- Minikube
- Docker
- Helm
- OpenTelemetry
- Microservices observability
- Prometheus metrics
- Grafana dashboards
- Jaeger distributed tracing
- Kubernetes Services
- Port forwarding
- Kubernetes troubleshooting
- Container resource management
- Helm deployment and troubleshooting

---

# 📚 Key Commands Summary

```bash
# Start Minikube
minikube start --driver=docker --cpus=4 --memory=10240

# Verify cluster
kubectl get nodes

# Add OpenTelemetry Helm repo
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

# Update repo
helm repo update

# Install OpenTelemetry Demo
helm install my-otel-demo open-telemetry/opentelemetry-demo -n default

# Check Helm release
helm list -n default

# Check pods
kubectl get pods -n default

# Check services
kubectl get svc -n default

# Access frontend
kubectl port-forward svc/frontend-proxy 8081:8080 -n default

# Access Grafana
kubectl port-forward svc/grafana 3000:80 -n default

# Uninstall demo
helm uninstall my-otel-demo -n default

# Delete Minikube
minikube delete
```

---

## 🎯 Project Result

Successfully deployed and explored the **OpenTelemetry Demo on a local Minikube Kubernetes cluster**, using Helm for application installation and Kubernetes port-forwarding for local access to the frontend and observability components.


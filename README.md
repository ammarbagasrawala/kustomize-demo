# kube-ammar demo

Step-by-step guide to build local Docker images, deploy them to a **kind-based local Kubernetes cluster** using **Kustomize**, and access the apps via port-forwarding.

---

## Prerequisites

Ensure the following tools are installed:

* Docker (CLI only is sufficient)
* kubectl (configured locally)
* kind (Kubernetes in Docker)
* kustomize (or kubectl ≥ 1.14, which embeds it)

Verify installations:

```sh
kubectl version --client
kind version
docker version
```

---

## 1) Create a local Kubernetes cluster

Create a kind cluster (one-time setup):

```sh
kind create cluster --name local-k8s
```

Verify cluster access:

```sh
kubectl cluster-info
kubectl get nodes
```

---

## 2) Build local Docker images

Build the two application images:

```sh
docker build -t app1:local ./app1
docker build -t app2:local ./app2
```

Load the images into the kind cluster:

```sh
kind load docker-image app1:local --name local-k8s
kind load docker-image app2:local --name local-k8s
```

> Note: kind nodes cannot see local Docker images unless they are explicitly loaded.

---

## 3) Deploy using Kustomize overlay

Apply the local overlay (this also creates the `ammars-cluster` namespace):

```sh
kubectl apply -k k8s/overlays/local
```

---

## 4) Verify resources

Check that pods and services are running:

```sh
kubectl get pods -n ammars-cluster
kubectl get svc -n ammars-cluster
```

You should see both `app1` and `app2` pods in `Running` state.

---

## 5) Port-forward services

Forward each service to a different local port (run in separate terminals):

```sh
kubectl port-forward svc/app1 8081:80 -n ammars-cluster
```

```sh
kubectl port-forward svc/app2 8082:81 -n ammars-cluster
```

---

## 6) Access the applications

Test the applications locally:

```sh
curl http://localhost:8081
curl http://localhost:8082
```

Expected responses:

* App 1 → `Hello from App 1`
* App 2 → `Hello from App 2`

---

## 7) Cleanup

Remove all deployed Kubernetes resources:

```sh
kubectl delete -k k8s/overlays/local
```

(Optional) Delete the kind cluster:

```sh
kind delete cluster --name local-k8s
```

---

## Notes

* This setup uses **local images only** (no registry required)
* Designed for local development and experimentation
* Easily extensible with Ingress, HPA, or environment-specific overlays

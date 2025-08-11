# Next.js Frontend Deployment with Docker and Kubernetes Autoscaling
This project demonstrates how to deploy a Next.js frontend application using Docker and Kubernetes, including setting up autoscaling with Horizontal Pod Autoscaler (HPA). It also includes stress testing to validate autoscaling behavior.

![home-page](public/home-from-docker.png)
## Project Overview
1. Built a Next.js frontend application.
2. Containerized the app using Docker.
3. Pushed the Docker image to Docker Hub.
4. Deployed the container on Kubernetes with a Deployment and Service.
5. Configured Horizontal Pod Autoscaler (HPA) for automatic scaling based on CPU utilization.
6. Used stress tools to generate load and observe scaling behavior.

## Docker Setup
### 1 Build Docker Image
```sh
docker build -t debjyoti08/taskflow:latest .
```
![docker-build](public/docker-build.png)

### 2 Run Docker Image Locally
```sh
docker run -p 3000:3000 debjyoti08/taskflow:latest
```
![docker-run](public/docker-run.png)
![home-page](public/home-from-docker.png)
### 3 Push The Image To DockerHub
```sh
docker push debjyoti08/taskflow:latest
```
![docker-push](public/docker-push.png)
![docker-hub](public/docker-hub.png)

## Kubernetes Deployment
### Start Minikube
```sh
minikube start
```
![minikube-start](public/minikube-start.png)

---
## Apply Kubernetes Manifests

### Namespace:
```sh
kubectl apply -f k8s/namespace.yaml
```
### Deployment:
```sh
kubectl apply -f k8s/deployment.yaml   
```
![deployment](public/apply-deployment.png)
### Service:
```sh
kubectl apply -f k8s/service.yaml
```
![service](public/apply-svc.png)
### HPA:
```sh
kubectl apply -f k8s/hpa.yaml
```
![hpa](public/apply-hpa.png)


### Verify Pods

```sh
kubectl get pods -n frontend-namespace
```
![pods](public/get-pods.png)

### Check Nodes
```sh
kubectl get nodes
```
![nodes](public/kubectl-nodes.png)

---

## Set Up Metrics Server and Horizontal Pod Autoscaler (HPA) on Minikube (Docker Driver)
## Deploy Metrics Server
Apply the official metrics-server components:
```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
## Patch Metrics Server Deployment to Fix TLS Issues
```sh
kubectl edit deployment metrics-server -n kube-system
```
### Under:
```sh
spec:
  template:
    spec:
      containers:
      - name: metrics-server
        args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s

```
### Add:
```sh
 - --kubelet-insecure-tls
```
![edit](public/edit-the-metrics-server.png)
### Save and exit.
---
## Restart Metrics Server to apply changes
```sh
kubectl rollout restart deployment metrics-server -n kube-system
```
## Verify Metrics Server is Running
```sh
kubectl get pods -n kube-system | grep metrics-server
```
You should see one pod running with 1/1 ready status.
![setup-metrics](public/setup-metrics-for-hpa.png)

## Check Metrics Availability
```sh
kubectl top nodes
kubectl top pods -n frontend-namespace
```
You should see CPU and memory usage metrics.
![setup-metrics](public/after-setup-metrics-hpa.png)
![resources-before-hpa](public/all-the-resources-before-hpa.png)
---
## Stress Testing
Used stress tools inside the pod to generate CPU load for testing autoscaling:
### Forward port to pod for access:
```sh
kubectl port-forward svc/frontend-service 8080:80 -n frontend-namespace
```
![port-forward](public/port-forward-svc.png)
![k8s-deploy-home](public/k8s-deploy-home.png)

### Before Load — Initial Service State
![all-the-resources-before-hpa](public/all-the-resources-before-hpa.png)

### Apply CPU load using stress command inside pod:
```sh
kubectl exec -it <pod-name> -n frontend-namespace -- stress --cpu 4 --timeout 30   
```
### Generating CPU load to trigger autoscaling.
![hpa-autoscal](public/hpa-autoscal.png)
![hpa-scalup](public/hpa-scalup.png)

### After Load — Service State During Scaling
![all-resource-after-scalup](public/all-resource-after-scalup.png)

### After Load — Autoscaling Back Down
![hpa-scaldown](public/hpa-scaldown.png)

 ### Autoscaler reducing replicas after load decreases.
![all-resource-after-scaldown](public/pods-auto-terminating.png)
![all-resource-after-scaldown](public/all-resource-after-scaldown.png)
---

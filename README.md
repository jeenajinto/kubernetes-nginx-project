# kubernetes-nginx-project
Kubernetes Learning
# Kubernetes & Minikube Setup Guide

Guide for setting up a local Kubernetes development environment using Minikube on Ubuntu.

## Table of Contents
- [Setup Environment](#setup-environment)
- [Install Minikube](#install-minikube)
- [Install kubectl](#install-kubectl)
- [Verification](#verification)
- [Kubernetes Resources](#kubernetes-resources)
- [Deployment Steps](#deployment-steps)
- [Troubleshooting](#troubleshooting)

## Setup Environment

Install required tools:

```bash
sudo apt update

sudo apt upgrade -y

sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo usermod -aG docker $USER
newgrp docker
```

Restart the EC2 instance, then start Minikube:

```bash
minikube start --driver=docker
```

## Install kubectl

```bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

## Verification

```bash
kubectl get pods -n kube-system
kubectl describe pod kube-apiserver-minikube -n kube-system
```

## Kubernetes Resources

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  WELCOME_MSG: "Hello from Kubernetes Nginx!"
```

Save as: `configmap.yaml`

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
stringData:
  ADMIN_PASSWORD: "MyS3cretPass"
```

Save as: `nginx-secret.yaml`

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Save as: `nginx-service.yaml`

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
        image: nginx:latest
        ports:
        - containerPort: 80
        env:
        - name: WELCOME_MSG
          valueFrom:
            configMapKeyRef:
              name: nginx-config
              key: WELCOME_MSG
        - name: ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: nginx-secret
              key: ADMIN_PASSWORD
```

Save as: `deployment.yaml`

## Deployment Steps

### 1. Apply ConfigMap

```bash
kubectl apply -f configmap.yaml
```

### 2. Apply Secret

```bash
kubectl apply -f nginx-secret.yaml
```

### 3. Apply Service

```bash
kubectl apply -f nginx-service.yaml
```

### 4. Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

### 5. Verify Deployment

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

## Troubleshooting

### Check Pod Status

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Check Service Status

```bash
kubectl get svc
kubectl describe svc nginx-service
```

### Access NGINX

```bash
kubectl port-forward svc/nginx-service 8080:80
# Visit http://localhost:8080
```

## Requirements

- Ubuntu-based Linux distribution
- Docker
- Minikube
- kubectl

## Notes

- The setup uses nginx as the example application
- ConfigMaps and Secrets are used for configuration management
- The deployment creates 3 replicas of the nginx pod

---

Created: 2026-02-12

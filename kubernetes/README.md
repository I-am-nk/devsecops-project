# Kubernetes Deployment for Tic Tac Toe

This directory contains Kubernetes manifests for deploying the Tic Tac Toe application.

## Components

1. **Deployment** - Manages the application pods with scaling and update strategies
2. **Service** - Exposes the application within the cluster
3. **Ingress** - Exposes the application to external traffic


# 🚀 DevSecOps Project — EC2 + kind + Argo CD Setup & Run Guide

This guide walks through setting up a local Kubernetes cluster using **kind** on an **Ubuntu EC2** instance, installing **Argo CD**, and exposing the UI and applications.

---

## 1. Prerequisites

- An **Ubuntu EC2** instance (18.04 / 20.04 / 22.04 recommended) with a **public IP**.
- An SSH user with `sudo` privileges (for example, `ubuntu`).
- A **GitHub Personal Access Token (PAT)** with `write:packages` and `read:packages` permissions for GHCR:
  - Create at: `GitHub → Settings → Developer settings → Personal access tokens`

---

## 2. EC2 Instance Preparation (Update + Basic Tools)

Update package index:

```
sudo apt update -y
```


---

## 3. Install Docker (Required by kind)

Install Docker:

```
sudo apt install docker.io -y
```

Allow your EC2 user to run Docker without `sudo` (replace `ubuntu` with your user if different):

```
sudo usermod -aG docker ubuntu
```


**Log out and log back in** (or `exec su - ubuntu`) so group changes take effect.

Verify Docker:

```
docker version
docker ps
```


---

## 4. Install kind and Create a Cluster

Install kind:

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

kind --version

or follow this documention:
https://kind.sigs.k8s.io/docs/user/quick-start/#installation


Create a cluster:

```
kind create cluster --name devsecops-project
```


---

## 5. Install kubectl

Download and install kubectl:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl

or follow this documention 
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/


Verify:

```
kubectl version --client --short
```


---

## 6. Install Argo CD

Create the Argo CD namespace and apply the official manifests:

```
kubectl create namespace argocd

kubectl apply -n argocd
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
``


Wait for Argo CD pods to become ready:


```
kubectl get pods -n argocd
```


Retrieve the **initial admin password**:

```
kubectl -n argocd get secret argocd-initial-admin-secret
-o jsonpath="{.data.password}" | base64 -d && echo
```

- Username: `admin`
- Password: value printed above

---

## 7. Port-Forward Argo CD UI

use the below command to forword the port for the ArgoCd

```
kubectl port-forward svc/argocd-server 9000:80 -n argocd --address 0.0.0.0
```


Now open the UI in your browser:

```
http://<EC2_PUBLIC_IP>:9000
```


> ⚠️ Keep this terminal session running. If your SSH session closes, the port-forward stops.  
> For security, restrict the EC2 security group rules to **your IP** when exposing these ports.

---

## 8. Create GHCR Image-Pull Secret (Docker Registry Secret)

```
kubectl create secret docker-registry github-container-registry
--docker-server=ghcr.io
--docker-username=${GHCR_USER}
--docker-password=${GHCR_PAT}
--docker-email=${EMAIL}
```

Replace your Github User Name , Email and Personal Access Token of Github above.


Attach this secret in your Deployment / StatefulSet using `imagePullSecrets` so Argo CD / Kubernetes can pull from GHCR.

---

## 9. Create Argo CD Application (Summary)

High-level steps (you can do this via UI or YAML):

1. Log in to Argo CD UI at `http://<EC2_PUBLIC_IP>:9000`.
2. Authenticate with:
   - Username: `admin`
   - Password: retrieved in step 6.
3. Create a new **Application**:

      Fill in details:

   - Application Name: `devsecops-project`
   - Project: `default`
   - Sync Policy: `Automatic`
   - Repository URL: https://github.com/I-am-nk/DevSecOps-Project.git
   - Revision: `HEAD`
   - Path: `Kubernetes`
   - Cluster URL: `https://kubernetes.default.svc`
   - Namespace: `default`

4. Click **Create**

Argo CD will sync your repo and deploy your app automatically 🎯

(If you already have an application manifest, you can also `kubectl apply -f app.yaml`.)

---

## 10. Access Your Deployed Application

### Option 1 — Port-Forward a Specific Pod (Quick, Not Permanent)

List pods:

```
kubectl get pods
```

Port-forward a pod
```
kubectl port-forward <pod-name> 3000:80 --address 0.0.0.0
```


Open in your browser:

http://<EC2_PUBLIC_IP>:3000




#congrates your app is live you will be able to see the GAME.
# Kubernetes CI/CD with Argo CD and Argo Rollouts

This guide provides a walkthrough for setting up a continuous deployment pipeline for a web application using Argo CD and Argo Rollouts on a Kubernetes cluster. Additionally, it covers the steps to containerize a React application using Docker.

## Prerequisites

Before you begin, ensure you have:

- A Kubernetes cluster
- `kubectl` configured to communicate with your cluster
- Docker installed for building and pushing your container image

## Setup

### 1. Install Argo CD

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

#### Apply the Argo CD Installation Manifest:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

#### Access The Argo CD API Server:

- Port-forward the Argo CD API server for local access:

  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```

- Open `https://localhost:8080` in your web browser.

#### Login Using CLI:

- Retrieve the auto-generated `admin` password:

  ```bash
  kubectl argocd admin initial-password -n argocd
  ```

- Login and update the password:

  ```bash
  argocd login localhost:8080
  argocd account update-password
  ```

### 2. Install Argo Rollouts

Argo Rollouts provides advanced deployment capabilities such as blue-green and canary deployments.

#### Install Argo Rollouts:

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml
```

#### Verify Installation:

```bash
kubectl get all -n argo-rollouts
```

## Containerize Your Web App

### Dockerfile

```Dockerfile
# Stage 1: Build the React application
FROM node:18-alpine as build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . ./
RUN npm run build

# Stage 2: Serve the React application from Nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Build and Push Your Container Image

```bash
docker build -t youtube-main .
docker tag youtube-main uniyalrachna/youtube-main:v1
docker push uniyalrachna/youtube-main:v1
```

## Deploy Your Application with Argo CD

```bash
argocd app create youtube-main --repo https://github.com/rachana-uniyal/argocd-assignment.git --path youtube --dest-namespace default --dest-server https://kubernetes.default.svc --directory-recurse --sync-policy auto
```

Verify the application deployment:

```bash
kubectl get application -n argocd
```

## Implement Argo Rollouts

### Add a new rollout:

```bash
kubectl apply -f https://raw.githubusercontent.com/rachana-uniyal/argocd-assignment/main/youtube/deployment.yaml
```

Monitor and promote the rollout:

```bash
kubectl argo rollouts get rollout youtube-main --watch
kubectl argo rollouts promote youtube-main
```

To revert all changes made during the setup and deployment process, follow these steps:

#### Delete the Argo CD Application:

```bash
argocd app delete youtube-main
```

#### Uninstall Argo CD and Argo Rollouts:

```bash
kubectl delete namespace argocd
kubectl delete namespace argo-rollouts
```


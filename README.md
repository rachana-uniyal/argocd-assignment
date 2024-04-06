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
